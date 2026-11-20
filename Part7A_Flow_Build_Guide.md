# Part 7A: Build the Two "Find Similar Jobs" Flows

**Agent:** Job Description Jarvis (Copilot Studio)
**Goal:** Build two agent flows that give the agent a search across all existing job descriptions:

| Flow | Input | Output | What it does |
|---|---|---|---|
| **JD Agent - Get Job Title Catalog** | none | `Catalog` (text) | Returns one line per job: `Job Code \| Title \| People Management` |
| **JD Agent - Get JD Details for Codes** | `JobCodes` (text) | `Jobs` (text, JSON) | Returns the full JD content for the requested job codes, **without** Job Code, Grade, Salary/Hourly, or Status |

Both flows run under your SharePoint connection, so managers never need access to the JD list.

---

## Before you start (5 minutes)

Have these ready:

1. **The SharePoint site URL** that holds your JD list.
2. **The JD list name**: the list your daily CSV import loads.
3. **Three real Job Codes** from the list, ideally a ladder (for example a Specialist I, II, and III), for testing Flow 2.
4. **The Job Code column's internal name.** You only need it if the dynamic content picker doesn't show the column in Flow 2. To find it:
   1. Open the JD list in SharePoint.
   2. Click **Settings** (gear) → **List settings**.
   3. Under **Columns**, click **Job Code**.
   4. In the browser address bar, read the text after `Field=`. That's the internal name, for example `JobCode` or `Job_x0020_Code`.
   5. Write it down. Repeat for any column that later comes back empty.

---

## Part 1: Create the topic that holds the flows

1. Open **Copilot Studio** → **Agents** → **Job Description Jarvis**.
2. Click **Topics** (top tabs).
3. Click **+ Add a topic** → **From blank**.
4. Click the topic name at the top-left (**Untitled**) and rename it to **`Find Similar Jobs`**.
5. Click **Save** (top-right).

You'll add both flows from inside this topic. That way each flow node, with its ID, shows up in the topic's code, which is what you'll send back in Part 4.

---

## Part 2: Flow 1, "JD Agent - Get Job Title Catalog"

### 2.1 Create the flow

1. In the **Find Similar Jobs** topic, under the **Trigger** node, click **+**.
2. Select **Add a tool** → **New Agent flow**.
3. The flow designer opens with two steps already in place:
   - **When an agent calls the flow** (trigger)
   - **Respond to the agent** (final step)
4. At the top-left, click the flow name and rename it to **`JD Agent - Get Job Title Catalog`**.
5. Leave the trigger with **no inputs**.

### 2.2 Add "Get items"

1. Hover between the trigger and **Respond to the agent** → click **+** → **Add an action**.
2. In the search box type **Get items** → select **SharePoint** → **Get items**.
3. If asked to sign in, use the same account your other JD flows use.
4. Fill in:
   - **Site Address:** select your JD site (or choose **Enter custom value** and paste the URL).
   - **List Name:** select your JD list.
5. **Turn on pagination:**
   1. Click the action's **Settings** tab (in the action panel).
   2. Turn **Pagination** **On**.
   3. **Threshold:** `5000`.

   Without this, only the first 100 jobs are returned.
6. **Optional, only inactive jobs exist in the list:**
   1. Click **Show all** / **Advanced parameters** → **Filter Query**.
   2. Enter `Status eq 'Active'`, replacing `Status` and `Active` with your column's internal name and its active value.

### 2.3 Add "Select" (one line per job)

1. Under **Get items**, click **+** → **Add an action**.
2. Search **Select** → choose **Data Operation** → **Select**.
3. **From:** click in the box → **Dynamic content** (lightning icon) → under **Get items** choose **body/value** (may be labeled **value**).
4. **Map:** click the small **T** icon (**Switch Map to text mode**) so Map becomes a single box.
5. Click in the Map box and build the line in this exact order:
   1. Insert dynamic content **Job Code** (under Get items).
   2. Type a space, a vertical bar, and a space: ` | `
   3. Insert dynamic content **Title**. If your list uses a separate **Job Title** column, use that one instead.
   4. Type ` | `
   5. Insert dynamic content **People Management**. If it's a choice column, pick **People Management Value**.
6. The Map box should now read like: `[Job Code] | [Title] | [People Management Value]`

### 2.4 Add "Join" (combine the lines)

1. Under **Select**, click **+** → **Add an action**.
2. Search **Join** → choose **Data Operation** → **Join**.
3. **From:** **Dynamic content** → under **Select** choose **Output**.
4. **Join with:**
   1. Click in the box → click **fx** (expression).
   2. Type: `decodeUriComponent('%0A')`
   3. Click **Add** / **OK**.

   This puts each job on its own line.

### 2.5 Return the catalog to the agent

1. Click the **Respond to the agent** step.
2. Click **+ Add an output** → choose **Text**.
3. **Name:** `Catalog`
4. **Value:** **Dynamic content** → under **Join** choose **Output**.

### 2.6 Save, test, publish

1. Click **Save** (top-right).
2. Click **Test** → **Manually** → **Test** → **Run flow** → **Done**.
3. Open the run:
   - **Get items:** expand **Outputs**. The `value` array should hold all your jobs (about 595).
   - **Join:** the output should be many lines like `ABC123 | Wire Operations Specialist II | Individual Contributor`.
   - **Respond to the agent:** `Catalog` holds the same text.
4. If prompted, click **Publish**.
5. Close the designer (back arrow) to return to the **Find Similar Jobs** topic. A flow node for **JD Agent - Get Job Title Catalog** is now on the canvas.
6. Click **Save** on the topic.

**Check:** if a line shows a blank where Title or People Management should be, go back to **Select** and re-insert that column's **Value** version.

---

## Part 3: Flow 2, "JD Agent - Get JD Details for Codes"

### 3.1 Create the flow

1. In the **Find Similar Jobs** topic, under the first flow node, click **+**.
2. Select **Add a tool** → **New Agent flow**.
3. Rename the flow (top-left) to **`JD Agent - Get JD Details for Codes`**.

### 3.2 Add the input

1. Click the trigger **When an agent calls the flow**.
2. Click **+ Add an input** → choose **Text**.
3. Name it **`JobCodes`**.
4. In the description box, enter: `Comma-separated job codes, for example ABC123,ABC124`

### 3.3 Add "Get items"

1. Under the trigger, click **+** → **Add an action**.
2. Search **Get items** → **SharePoint** → **Get items**.
3. **Site Address:** the same JD site as Flow 1.
4. **List Name:** the same JD list.
5. **Settings** tab → **Pagination** **On** → **Threshold** `5000`.
6. **Optional:** the same `Status eq 'Active'` Filter Query as Flow 1, if you used one.

### 3.4 Add "Filter array" (keep only the requested codes)

1. Under **Get items**, click **+** → **Add an action**.
2. Search **Filter array** → **Data Operation** → **Filter array**.
3. **From:** **Dynamic content** → under **Get items** choose **body/value**.
4. **Left value (first box):**
   1. Click in the box → **fx**.
   2. Type: `concat(',', replace(triggerBody()?['text'], ' ', ''), ',')`
   3. Click **Add**.

   `triggerBody()?['text']` is the **JobCodes** input. If the editor says it's invalid:
   1. Delete `triggerBody()?['text']` from the expression.
   2. With the cursor in its place, switch to the **Dynamic content** tab and insert **JobCodes**.
   3. Click **Add**.
5. **Operator (middle dropdown):** **contains**
6. **Right value (third box):**
   1. Click in the box → **fx**.
   2. Type: `concat(',', ` (with a trailing space).
   3. Switch to the **Dynamic content** tab → insert **Job Code** (under Get items).
   4. Switch back to the expression and finish it with: `, ',')`

      The full expression looks like `concat(',', item()?['JobCode'], ',')`.
   5. Click **Add**.

   If **Job Code** isn't offered, type the expression by hand using the internal name you wrote down, for example `concat(',', item()?['JobCode'], ',')`.

The commas on both sides make sure `ABC12` matches only `ABC12`, never `ABC123`.

### 3.5 Add "Select" (shape each job, restricted fields left out)

1. Under **Filter array**, click **+** → **Add an action**.
2. Search **Select** → **Data Operation** → **Select**.
3. **From:** **Dynamic content** → under **Filter array** choose **Body**.
4. **Map:** stay in key/value mode (two columns). Add these 11 rows. Type the **Key** exactly as shown. For each **Value**, insert the column from dynamic content (under **Filter array**, or **Get items** if that's where it appears). For choice columns, pick the **Value** version.

| # | Key (type exactly) | Value (dynamic content) |
|---|---|---|
| 1 | `title` | Title (or Job Title) |
| 2 | `people_management` | People Management (Value) |
| 3 | `sales` | Sales/Non-Sales (Value) |
| 4 | `relationship_manager` | Relationship Manager (Value) |
| 5 | `nmls` | NMLS Required (Value) |
| 6 | `purpose` | Purpose |
| 7 | `principal_duties` | Principal Duties and Responsibilities |
| 8 | `education` | Education Requirements |
| 9 | `work_experience` | Work Experience Requirements |
| 10 | `ksas` | Knowledge Skills Abilities |
| 11 | `certifications` | Certifications/Licenses |

> **Do not add** Job Code, Grade, Salary/Hourly, or Status. The comparison works by title, so restricted fields never reach the prompt or the manager.

**If a column isn't in the dynamic content list:** click the Value box → **fx** → type `item()?['InternalName']` using that column's internal name (for a choice column: `item()?['InternalName']?['Value']`) → **Add**.

### 3.6 Return the jobs to the agent

1. Click the **Respond to the agent** step.
2. Click **+ Add an output** → **Text**.
3. **Name:** `Jobs`
4. **Value:**
   1. Click in the box → **fx**.
   2. Type: `string(body('Select'))`

      If your Select step has a different name (for example **Select 1**), use that name with spaces as underscores: `string(body('Select_1'))`. Check the step's title in the designer.
   3. Click **Add**.

### 3.7 Save, test, publish

1. Click **Save**.
2. Click **Test** → **Manually** → **Test**.
3. In **JobCodes**, enter your three real job codes separated by commas, for example `ABC101,ABC102,ABC103`.
4. Click **Run flow** → **Done**, then open the run.
5. Check:
   - **Filter array:** **Outputs** has exactly **3** items.
   - **Select:** **Outputs** shows 3 objects, each with the 11 keys.
   - **Respond to the agent:** `Jobs` is a JSON list of 3 jobs with **no** grade, salary/hourly, status, or job code.
6. Run one more test with a code that doesn't exist, for example `ZZZ999`. **Filter array** should return **0** items, and `Jobs` should be `[]`.
7. If prompted, click **Publish**.
8. Close the designer to return to the **Find Similar Jobs** topic. A second flow node now sits under the first.
9. Click **Save** on the topic.

---

## Part 4: Send the two flow nodes back

1. In the **Find Similar Jobs** topic, click **…** (top-right of the canvas) → **Open code editor**.
2. Find the two blocks that start with `- kind: InvokeFlowAction`.
3. Copy **both blocks completely**, from `- kind: InvokeFlowAction` through each block's `flowId:` line.
4. Paste them into the chat.

These blocks contain each flow's ID and the exact input and output names Copilot Studio assigned (for example `text`, `catalog`, `jobs`). I'll use them to write the finished **Find Similar Jobs** topic code and the two prompts (**Shortlist Similar Jobs** and **Compare With Existing Jobs**) as Part 7B.

Don't worry about wiring the flow nodes' inputs or outputs on the canvas. The final topic code will replace them.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Get items returns only 100 rows | Pagination is off | **Settings** → Pagination **On**, Threshold `5000` |
| A catalog line shows `\| \|` gaps | A choice or lookup column was inserted as an object | Re-insert that column's **Value** version |
| Filter array returns 0 rows for real codes | Spacing or case differs from the list | Copy the codes exactly from the list; the flow already strips spaces |
| Filter array expression error | `triggerBody()?['text']` doesn't match the input name | Replace it with **JobCodes** from dynamic content |
| `Jobs` output is empty but Select shows items | `body('Select')` points to the wrong step name | Use the exact step name, e.g. `body('Select_1')` |
| A Select value is blank for every job | Wrong column or internal name | Check the internal name in **List settings** (`Field=` in the URL) and use `item()?['InternalName']` |
| Flow fails with a connection error when the agent runs it | The connection isn't shared with the agent's flow run | Open the flow → **Details** → make sure the SharePoint connection is your (service) account's and the flow is published |

---

## Done checklist

- [ ] Topic **Find Similar Jobs** exists and is saved.
- [ ] **JD Agent - Get Job Title Catalog** returns about 595 lines in `Catalog`.
- [ ] **JD Agent - Get JD Details for Codes** returns exactly the requested jobs in `Jobs`, with no Job Code, Grade, Salary/Hourly, or Status.
- [ ] A non-existent code returns `[]`.
- [ ] Both flows are published.
- [ ] Both `InvokeFlowAction` blocks have been copied into the chat.
