# 🏫 NYSDOE: Agentic AI for School District Automation

## Table of Contents

- [Use Case Description](#use-case-description)
- [Pre-requisites](#pre-requisites)
  - [Step A: Set up the NYC Location API connection](#step-a-set-up-the-nyc-location-api-connection)
  - [Step B: Import the NYC Location API tool](#step-b-import-the-nyc-location-api-tool)
  - [Step C: Import the mock Security and Facilities API tools](#step-c-import-the-mock-security-and-facilities-api-tools)
- [Part 1 — Building the Operations Workflow (Use Case 1)](#part-1--building-the-operations-workflow-use-case-1)
  - [Step 1: Create the relocation_flow and configure inputs and outputs](#step-1-create-the-relocation_flow-and-configure-inputs-and-outputs)
  - [Step 2: Add the get_district_info tool node](#step-2-add-the-get_district_info-tool-node)
  - [Step 3: Add the book_rooms tool node](#step-3-add-the-book_rooms-tool-node)
  - [Step 4: Add the draft_output prompt node](#step-4-add-the-draft_output-prompt-node)
  - [Step 5: Map the prompt inputs](#step-5-map-the-prompt-inputs)
  - [Step 6: Connect the output and save](#step-6-connect-the-output-and-save)
- [Part 1B — Building the Emergency Response Workflow (Use Case 2)](#part-1b--building-the-emergency-response-workflow-use-case-2)
  - [Step 1: Create the threat_response_flow and configure inputs and outputs](#step-1-create-the-threat_response_flow-and-configure-inputs-and-outputs)
  - [Step 2: Add the get_district_info tool node](#step-2-add-the-get_district_info-tool-node-1)
  - [Step 3: Add the alert_principals tool node](#step-3-add-the-alert_principals-tool-node)
  - [Step 4: Add the set_office_hvac tool node](#step-4-add-the-set_office_hvac-tool-node)
  - [Step 5: Add the lock_office_doors tool node](#step-5-add-the-lock_office_doors-tool-node)
  - [Step 6: Add the draft_report prompt node](#step-6-add-the-draft_report-prompt-node)
  - [Step 7: Map the prompt inputs and connect the output](#step-7-map-the-prompt-inputs-and-connect-the-output)
- [Part 2 — Building Agents and Wiring Everything Together](#part-2--building-agents-and-wiring-everything-together)
  - [Create the Operations Agent](#create-the-operations-agent)
  - [Create the Emergency Response Agent](#create-the-emergency-response-agent)
  - [Build and Connect the School Research Assistant (watsonx.ai)](#build-and-connect-the-school-research-assistant-watsonxai)
  - [Create the Orchestrator Agent](#create-the-orchestrator-agent)
- [End-to-End Testing](#end-to-end-testing)
- [Pulling it all together](#pulling-it-all-together)

---

## Use Case Description

The New York State Department of Education manages hundreds of school buildings across dozens of districts. When disruptions occur — a broken water main, a severe weather event, a facilities emergency — district coordinators must manually look up buildings, identify capacity, coordinate relocations, draft notifications, and execute emergency protocols. This process is slow and error-prone.

In this lab you will build a network of AI agents that automates this entire process end-to-end. You will create two **agentic workflows** (flows) directly in the Watson Orchestrate UI:

- **Relocation Flow** — automatically finds an available school, books rooms, and drafts parent and staff notification emails when a building is disrupted
- **Threat Response Flow** — automatically fetches all buildings in a district, bulk-alerts school principals, switches office HVAC to low power, locks office doors, and produces a full incident report when a weather or security threat is declared

You will then build specialist agents backed by those flows, connect an external research agent, and wire everything together under a central **Orchestrator Agent** that routes incoming district triggers to the right specialist automatically.

---

## Pre-requisites

You will set up the connection and import all required tools yourself through the Watson Orchestrate UI. Follow Steps A, B, and C below before starting Part 1.

---

### Step A: Set up the NYC Location API connection

The Operations and Emergency Response flows call the **real NYC Department of Education Location API** to look up schools (`schoolDBN`) and offices (`locationCode`) in a district. This API requires an OAuth2 connection configured with shared credentials.

1. In Watson Orchestrate, click on the hamburger menu to open up the left navigation panel. Next, under **Manage**, click **Security**.

![alt text](../Screenshots/clickhamburger.png)
![alt text](../Screenshots/managesecurity.png)

2. Click on the **Connections** tab, then click on the **Add connection +** box.

![alt text](../Screenshots/connectionstab.png)

3. In the **Connection ID** field, enter `location_api` as the connection name. Enter `NYS DOE Location API` as the **Display Name**. You may also optionally add a description field. Click **Save and continue** at the bottom right. A confirmation message will pop up and ask you if you want to continue, you can go ahead and click **Continue**.

![alt text](../Screenshots/setupconnection.png)

4. In the **Configure draft connection page**, leave **Signle sign-on(SSO)** off. Under **Authentication type**, select **OAuth 2.0 — Client Credentials**. 

![alt text](../Screenshots/configureconnection.png)


5. Fill in the following fields, you can leave the other fields as is. Then, scroll down to select **Team credentials** as the credential type, click **Next** to continue:

   | Field | Value |
   |---|---|
   | **Token URL** | `https://apistg.schools.nyc/doe/stg/v1/oauth/oauth2/token` |
   | **Client ID** | `Ask your instructor` |
   | **Client Secret** | `Ask your instructor` |
   | **Scope** | `location` |

![alt text](../Screenshots/fillincreds.png)

![alt text](../Screenshots/credential_type.png)

6. In the **Configure live connection** page, you can click **Paste draft configuration**. It will automatically populate the values you filled in for the draft configuration.

![alt text](../Screenshots/pastedraft.png)

7. Click **Finish**

![alt text](../Screenshots/finishconnect.png)

> ✅ You should see the connection status change to **Connected**. If it shows an error, double-check the Client ID and Secret — make sure there are no extra spaces.

---

### Step B: Import the necessary tools (NYC Location API tool, Security API tool, and Facilities API tool) 

Your instructor will provide you with three yaml files for each of these tools. If you don't have these tools handy yet, please ask an instructor. 

The NYS Location API tool calls the real NYSDOE Location API and returns the list of schools (`schoolDBN`) and offices (`locationCode`) for a district. The Security and Facilities APIs are mock services that simulate door locking, HVAC control, room booking, and principal alerts. They accept the same `schoolDBN` and `locationCode` values returned by the real Location API.

The file names are as follows:
nysdoe_location_api.yaml
nysdoe_facilities_api.yaml
nysdoe_security_api.yaml


1. In Watson Orchestrate, click on the hamburger menu in the top left corner, and click **Build** in the left navigation panel.

![alt text](../Screenshots/buildimporttools.png)

2. Navigate to the **All Tools** tab, and click **Create tool**.

![alt text](../Screenshots/alltoolscreate.png)

3. Click **OpenAPI** to import your OpenAPI yaml files. On the **Upload files** page, either drag and drop your file or click to upload the yaml file you recieved from your instructor. Then, click **Next**:

![alt text](../Screenshots/selectopenapi.png)
![alt text](../Screenshots/clickuploadtool.png)


4. On the **Operations** page, click the topmost check box to select all tools. Then, click **Next**.

![alt text](../Screenshots/selectalltools.png)

5. On the **Connections** page, select the connection you made in **Step A**. (It should be named NYS DOE Location API). Click **Done**.

![alt text](../Screenshots/selectconnection.png)

6. Repeat the above steps for each yaml file. **Note:** the security and factilities yamls contain multiple tools, so make sure you select all tools. Security and Facilties yaml upload will **not** require a connection set up.

7. ✅ After importing all 3 files confirm the following tools appear in your tool list:

   - `getDistrictAdminDetails` *(Location API — real)*
   - `bookBuilding` *(Facilities API — mock)*
   - `bulkSetHVAC` *(Facilities API — mock)*
   - `setDoorState` *(Security API — mock)*
   - `bulkAlertPrincipals` *(Security API — mock)*

If any are missing, ask your instructor before proceeding.

---
## Part 1 — Building the Operations Use Case - "Automated District Relocation"

> **Scenario:** A water main has broken at a middle school in District 4. The Operations workflow will automatically find a relocation building, book rooms, and draft parent and staff notification emails.

### The flow we are building:

![alt text](../Screenshots/operationsflow.png)

We will now walk through creating the above workflow step by step.

---

### Step 1: Create the Operations Agent

1. Open the Agent Builder in Watson Orchestrate by clicking **Build** from the lefthand navigation menu.

![alt text](../Screenshots/buildimporttools.png)

2. Click **Create Agent**, then select **Create from scratch**.

Name the agent:
```
operations_agent
```

Give it the following description:
```
Handles NYSDOE district-level facility disruptions. Given a disruption trigger (water main break, power outage, building closure), this agent finds available relocation buildings, books rooms, and drafts parent and staff notification emails.
```

Then, click **Create**

![alt text](../Screenshots/opagentname.png)

4. Scroll down to the **Behavior** (Instructions) section and enter:

```
You are the NYSDOE Operations Agent. When a facility disruption is reported,
invoke the relocation_flow tool immediately — do not ask questions first.

Extract these inputs from the message:
- district_id, affected_building_name, affected_buildingCode, disruption_type, rooms_needed, start_date

affected_buildingCode is the schoolDBN of the disrupted building (e.g. "M057"). It is used to
prevent that building from being booked as its own relocation site. If the user does not provide
it, ask for it once before invoking the flow.

If district_id cannot be determined, ask for it once before invoking the flow.

Present the full flow output to the user (booking confirmation, relocation plan,
parent email, staff email), then ask:
"Would you like to send these notifications? Reply YES to confirm or NO to discard."

If no suitable building is found, report it clearly and suggest escalating to the Orchestrator.
```

![alt text](../Screenshots/opagentbehavior.png)


### Step 2: Create the Operations Workflow

Now we will create the Operations Workflow within our Operations Agent. 

1. Scroll down to the **Toolset** section and click **Add Tool**:

![alt text](../Screenshots/toolsetaddtool.png)

2. Select **Agentic workflow**:

![alt text](../Screenshots/createagenticworkflow.png)

3. The flow editor will open and prompt you to name your agentic workflow. You can also click edit details to update the Name and Description fields. Then clikc **Done** to save.


Change the name to:
```
relocation_flow
```

Change the description to:
```
Finds available relocation buildings in a district, books rooms, and drafts parent and staff notification emails.
```

![alt text](../Screenshots/renameflow.png)

4. Click on the green "play" button to open the **Inputs**. Click **Add** to add input variables to your workflow. 

![alt text](../Screenshots/addinputs.png)

5. Select the variable type **String** and name your variable **district_id**. Make sure the **Required** field is toggled **ON**. Then, click **Add**

![alt text](../Screenshots/selectvariabletype.png)

![alt text](../Screenshots/namevariableadd.png)

6. Repeat this process for the following variables, making sure to set the correct variable type and to toggle required for each one.

| Variable name | Type | Required | Description |
|---|---|---|---|
| `district_id` | String | Yes | Numeric district ID, e.g. `"4"` |
| `affected_building_code` | String | Yes | schoolDBN of the disrupted building, e.g. `"M057"` — prevents it from being booked as its own relocation site |
| `disruption_type` | String | Yes | e.g. `"water main break"` |
| `rooms_needed` | Integer | Yes | Number of rooms needed |
| `start_date` | String | Yes | Format: `YYYY-MM-DD` |

7. ✅ If you click on the green play button to view all inputs, it should look something like this: 

![alt text](../Screenshots/allinputsoperations.png)

8. Click on the dashed box labeled **Add your first step +** and click **Call a tool**. 

![alt text](../Screenshots/addfirststepcalltool.png)

9. A tools page will open on the left side of the screen. In the search bar, type in `getDistrictAdminDetails` and drag and drop it to the flow:

![alt text](../Screenshots/getdistrictadmindrag.png)

10. Hover over the arrow connecting the "Get schools and offices" (added in the previous step) to the End node and click **+**. Select **Call a tool**.

![alt text](../Screenshots/hoveroverget.png)

11. In the **Tools** search bar, search for `bookBuilding`. Same as before, drag and drop it below the "Get schools and offices" tool before the output/end node.

![alt text](../Screenshots/bookbuildingdrag.png)

12. Hover over the arrow connecting "Book rooms in a building" to the End node and click **+**. Select **Add a flow activity** and then click **Generative prompt**:

![alt text](../Screenshots/addflowgenprompt.png)

13. In the **System prompt** enter in the following:

```
You are the NYSDOE Operations Agent. You produce professional, clear district 
communications and operational plans. Always fill every field with real values 
— never use placeholders.
```

![alt text](../Screenshots/operationsystemprompt.png)

14. In the **User prompt** enter in the following:

```
A facility disruption has occurred. Using the details below, produce the full relocation output.

DISRUPTION DETAILS:
- Affected building: {flow.input.affected_building_code}
- District: {flow.input.district_id}
- Disruption type: {flow.input.disruption_type}
- Effective date: {flow.input.start_date}

BOOKING CONFIRMATION (from API):
- Booking ID: {flow["Book rooms in a building for relocation"].output.booking_id}
- Relocation building: {flow["Book rooms in a building for relocation"].output.buildingName}
- Relocation building code: {flow["Book rooms in a building for relocation"].output.buildingCode}
- Rooms booked: {flow["Book rooms in a building for relocation"].output.rooms_booked}

Produce the following four sections, clearly separated.
IMPORTANT: The affected building ({flow.input.affected_building_code}) is being EVACUATED.
The relocation building ({flow["Book rooms in a building for relocation"].output.buildingName})
is the NEW TEMPORARY SITE. Never confuse the two.

## ✅ BOOKING CONFIRMED
A short confirmation (3–4 lines): state the affected building being evacuated,
the new temporary site name and address, booking ID, rooms booked, and effective date.

## 📋 RELOCATION PLAN
A structured operational plan for the operations team. Include:
1. Immediate actions (Day 0 — {flow.input.start_date})
2. Logistics (Day 1): room setup, IT transfer, student records, meal service
3. Communications: confirm emails sent, update district website
4. Responsible parties: assign each step to a role
5. Return plan trigger: define the condition to return to {flow.input.affected_building_code}

## 📧 PARENT EMAIL
Subject: Important: School Relocation Notice — {flow.input.affected_building_code}

A full professional parent notification email. Include what happened, the new
temporary site name and full address, effective date, next steps, and contact info.

## 📧 STAFF EMAIL
Subject: STAFF NOTICE: Facility Relocation — {flow.input.affected_building_code}

A full professional staff notification email. Include the disruption detail,
new site name and address, booking confirmation ID, facilities notes, and contact info.
```

![alt text](../Screenshots/operationuserprompt.png)

Once you are satisfied with the prompts, you can close the prompt settings.

15. Finally, click **Done** at the top left hand corner to close the flow editor. Your completed flow should look like this:

![alt text](../Screenshots/operationsflow.png)

---

### Step 3: Test the Operations Agent

1. In the right hand agent chat preview page, send the following message to the chat box and hit **Send**:

```
A water main broke at M007 in District 4. We need to relocate immediately.
The disruption started today and we need 8 rooms starting 2025-07-15.
```

![alt text](../Screenshots/operationsmessage.png)

✅ The agent should call the `relocation_flow` and return a booking confirmation, relocation plan, and two draft emails. You can click on the **Show Reasoning** tab to open up the step by step reasoning process.

![alt text](../Screenshots/operationsoutput1.png)
![alt text](../Screenshots/operationsoutput2.png)

✅ The `relocation_flow` is built and the `operations_agent` is ready.

---

## Part 2 — Building the Emergency Use Case - "District-Wide Threat Response"

> **Scenario:** A tornado watch has been issued for District 7. The Emergency Response workflow will automatically fetch all buildings in the district, bulk-alert school principals by SMS, switch office HVAC to low power, lock office doors, and produce a full incident report.

### The flow we are building:

![alt text](../Screenshots/emergencyflow.png)

We will now walk through creating the above workflow step by step.

---

### Step 1: Create the Emergency Response Agent

1. In the **Agent Builder** (Click on the hamburger menu, click Build), click **Create Agent** → **Create from scratch**.

![alt text](../Screenshots/createagent.png)

Name the agent:
```
emergency_response_agent
```

Give it the following description:
```
District-Wide Threat Response Agent. Triggered by a severe weather warning for a specific geographic district. Differentiates response by building type: for schools it bulk-alerts every principal by SMS; for office buildings it switches HVAC to low-power mode and remotely locks all doors. Returns a full timestamped incident report.
```

Then, click **Create**.

![alt text](../Screenshots/namedescemergency.png)

2. Scroll down to the **Behavior** (Instructions) section and enter:

```
You are the NYSDOE Emergency Response Agent for district-wide threat response.

When a weather warning is reported, collect the district_id, threat_type, and
severity from the user, then call threat_response_flow immediately.
That flow handles all response steps — fetching district buildings, alerting
school principals, securing office HVAC, locking office doors, and drafting
the incident report.

Present the returned incident report clearly to the user. If any principals
were skipped or any office actions need verification, highlight them so the
operator can follow up.
```

![alt text](../Screenshots/emergencyagentbehavior.png)

---

### Step 2: Create the Emergency Response Workflow

Now we will create the Emergency Response Workflow within our Emergency Response Agent.

1. Scroll down to the **Toolset** section and click **Add Tool**:

![alt text](../Screenshots/toolsetaddtoolsemergency.png)

2. Select **Agentic workflow**:

![alt text](../Screenshots/createagenticworkflow.png)

3. The flow editor will open and prompt you to name your agentic workflow. You can also lick **edit details** to update the Name and Description fields, then click **Done** to save.

![alt text](../Screenshots/createagenticworkflow.png)

Change the name to:
```
threat_response_flow
```

Change the description to:
```
Finds district buildings, bulk-alerts school principals, secures office HVAC and doors, and drafts a full incident report.
```

![alt text](../Screenshots/renameemergencyflow.png)

4. Click on the green "play" button to open the **Inputs**. Click **Add** to add input variables to your workflow.

![alt text](../Screenshots/addinputsemer.png)

5. The same way you did for the Operations Flow, add the following input variables, making sure to set the correct variable type and toggle Required for each one:

| Variable name | Type | Required | Description |
|---|---|---|---|
| `district_id` | String | Yes | Affected district, e.g. `"7"` |
| `threat_type` | String | Yes | e.g. `"tornado watch"` |
| `severity` | String | Yes | e.g. `"high"` |
| `alert_message` | String | No | Optional custom SMS override text |

**Note:** Make sure you toggle required **OFF** for `alert_message`

The Inputs page should look like this when you're done:

![alt text](../Screenshots/allinputsemergency.png)

6. Click on the dashed box labeled **Add your first step +** and click **Call a tool**.

![alt text](../Screenshots/addfirststepcalltoolemer.png)

7. In the **Tools** search bar, type `getDistrictAdminDetails` and drag and drop it onto the flow:

![alt text](../Screenshots/getdistrictadmindragemer.png)

> 💡 This calls the real NYC Location API. The response gives you `schools[].schoolDBN` (school building codes) and `offices[].locationCode` (office building codes). The next nodes use these to alert principals and secure the superintendent office.

8. Hover over the arrow after the "Get schools and offices" node and click **+**. Select **Call a tool**, search for `bulkAlertPrincipals`, and drag and drop it onto the flow:

![alt text](../Screenshots/bulkalergdrag.png)

9. Hover over the arrow after the "Alert ALL school principals" node and click **+**. Select **Call a tool**, search for `bulkSetHVAC`, and drag and drop it onto the flow:

![alt text](../Screenshots/bulkhvacdrag.png)

> 💡 `bulkSetHVAC` targets **all buildings in the district** by `districtCode` in a single API call. The response includes `buildings_count` so the prompt can report exactly how many offices were secured.

10. Hover over the arrow after the "Set HVAC mode" node and click **+**. Select **Call a tool**, search for `bulksetdoors`, and drag and drop it onto the flow:

![alt text](../Screenshots/setdoorstatedrag.png)

11. Hover over the arrow after the "Lock or unlock" node and click **+**. Select **Add a flow activity** then click **Generative prompt**:

![alt text](../Screenshots/addflowgenpromptemer.png)

12. In the **System prompt** enter the following:

```
You are the NYSDOE Emergency Response Agent. Produce a concise, operational
incident report using only the values provided. Never use placeholders.
```

![alt text](../Screenshots/emergencysystemprompt.png)

13. In the **User prompt** enter the following:

```
A severe weather warning has triggered a district-wide threat response.
Using the values below, produce a complete incident report.

THREAT DETAILS:
- District: {flow.input.district_id}
- Threat type: {flow.input.threat_type}
- Severity: {flow.input.severity}

PRINCIPAL ALERT RESULTS (school buildings):
- Principals alerted: {flow["Alert ALL school principals in a district"].output.alerted_count}
- Principals skipped (no phone on record): {flow["Alert ALL school principals in a district"].output.skipped_count}

OFFICE ACTION RESULTS (all office buildings in district):
- Bulk HVAC status: {flow["Set HVAC mode for ALL buildings in a district"].output.status}
- Door lock status: {flow["Lock or unlock ALL buildings in a district"].output.status}

Produce the following three sections, clearly separated.

## INCIDENT SUMMARY
Write a short operational summary for District {flow.input.district_id} describing
the {flow.input.threat_type} threat at severity {flow.input.severity}. Confirm that
principal SMS alerts were sent, that HVAC was bulk-set to low_power across all office
buildings in the district, and that office doors were locked.

## ACTION LOG
List every completed action as a bullet:
• bulkAlertPrincipals — {flow["Alert ALL school principals in a district"].output.alerted_count} school principals alerted, {flow["Alert ALL school principals in a district"].output.skipped_count} skipped
• bulkSetHVAC — {flow["Set HVAC mode for ALL buildings in a district"].output.buildings_count} office buildings set to low_power — status: {flow["Set HVAC mode for ALL buildings in a district"].output.status}
• setDoorState — superintendent office doors locked — status: {flow["Lock or unlock ALL buildings in a district"].output.status}

## FOLLOW-UP
List immediate follow-up steps for the district coordinator.
If {flow["Alert ALL school principals in a district"].output.skipped_count} > 0, note that those principals require manual outreach.
If {flow["Set HVAC mode for ALL buildings in a district"].output.status} is not 'bulk_updated', flag the HVAC action for manual verification.
If {flow["Lock or unlock ALL buildings in a district"].output.status} is not 'updated', flag the door lock for manual verification.
```

![alt text](../Screenshots/emergencyuserprompt.png)

Once you are satisfied with the prompts, close the prompt settings.

14. Finally, click **Done** at the top left hand corner to close the flow editor. Your completed flow should look like this:

![alt text](../Screenshots/emergencyflow.png)

---


## Part 3 — Building the Orchestrator Agent

Both specialist agents are already built from Parts 1 and 2. In this part you will create the **Orchestrator Agent** and wire the two specialist agents to it as collaborators.

---

### Step 1: Create the Orchestrator Agent

1. In the **Agent Builder**, click **Create Agent** → **Create from scratch**.

Name the agent:
```
orchestrator_agent
```

Give it the following description:
```
Central NYSDOE Orchestrator Agent. Triages every incoming district trigger and routes it to the correct specialist: facility disruptions go to the Operations Agent, weather or security threats go to the Emergency Response Agent.
```

Then, click **Create**.

![alt text](../Screenshots/orchestratoragentname.png)

2. Scroll down to the **Behavior** (Instructions) section and enter:

```
You are the NYSDOE Orchestrator Agent. Your sole job is to triage incoming
triggers and delegate to the appropriate specialist agent. Never attempt to
handle facility operations or emergency protocols yourself.

ROUTING RULES:

1. FACILITY DISRUPTION (water main break, power outage, building closure,
   structural damage, HVAC failure, relocation needed)
   → Route to: operations_agent
   Pass: district_id, affected_building_code, disruption_type, rooms_needed, start_date.

2. WEATHER OR SECURITY THREAT (tornado watch, ice storm, hurricane, severe
   weather warning, lockdown)
   → Route to: emergency_response_agent
   Pass: district_id, threat_type, and severity.

BEHAVIOUR:
- Extract all context from the message before routing.
- If district_id is missing, ask for it once — then route immediately.
- Do not ask unnecessary clarifying questions.
- Present the specialist agent's full output without summarising or truncating.
- If a specialist agent fails, report the failure clearly.
```

![alt text](../Screenshots/orchestratoragentbehavior.png)

---

### Step 2: Add the Specialist Agents as Collaborators

1. Scroll down to the **Agents** section (separate from Toolset). Click **Add Agent**:

![alt text](../Screenshots/addagent.png)

2. Search for and add the **Operations Agent**:

![alt text](../Screenshots/addoperationsagent.png)

3. Repeat and add the **Emergency Response Agent**:

![alt text](../Screenshots/addemergencyagent.png)

✅ Both specialist agents should now appear as collaborators under the Agents section:

![alt text](../Screenshots/bothagentsadded.png)

4. Click **Save**.

---


### Step 3: Test the Emergency Response Agent

1. In the right hand agent chat preview page, send the following message to the chat box and hit **Send**:

```
A tornado watch has been issued for District 7. Severity is HIGH.
Take all necessary emergency actions across all buildings in the district.
```

![alt text](../Screenshots/emergencymessage.png)

✅ The agent should call the `threat_response_flow tool` and return an incident report with an INCIDENT SUMMARY, ACTION LOG showing the new HVAC and door status values, and a FOLLOW-UP section. You can click on the **Show Reasoning** tab to open up the step by step reasoning process.

![alt text](../Screenshots/emergencyoutput1.png)
![alt text](../Screenshots/emergencyoutput2.png)

Click **Save**.

✅ The `threat_response_flow` is built and the `emergency_response_agent` is ready.

---

### Step 3: Test the Orchestrator Agent

1. In the right hand agent chat preview page, send the following message to test **Use Case 1 — Facility Relocation**:

```
A water main broke at M007 in District 4. We need to relocate immediately.
The disruption started today and we need 8 rooms starting 2025-07-15.
```

![alt text](../Screenshots/orchestratormessage1.png)

✅ **Expected flow:** Orchestrator → Operations Agent → `relocation_flow` → booking confirmation + relocation plan + parent email + staff email

![alt text](../Screenshots/orchestratoroutput1.png)

2. Now send the following message to test **Use Case 2 — Emergency Threat Response**:

```
A tornado watch has been issued for District 7. Severity is HIGH.
Take all necessary emergency actions across all buildings in the district.
```

![alt text](../Screenshots/orchestratormessage2.png)

✅ **Expected flow:** Orchestrator → Emergency Response Agent → `threat_response_flow` → incident report showing HVAC and door status values

![alt text](../Screenshots/orchestratoroutput2.png)

Click **Save**.

✅ The `orchestrator_agent` is built and routing correctly to both specialist agents.

---

## Part 4 — Add an External LangGraph Agent from watsonx.ai *(Optional)*

In this optional section you will build a **School Research Assistant** as a LangGraph agent directly inside **watsonx.ai**, deploy it as a hosted endpoint, and connect it to Watson Orchestrate so the Orchestrator can call it for research and policy lookups.

---

### Step 1: Create the LangGraph Agent in watsonx.ai

1. Open a new browser tab and navigate to your watsonx.ai instance. In the left navigation click **Projects**, then open the lab project provided by your instructor.

2. Inside the project click **New asset** → **AI Agent**:

![alt text](../Screenshots/wxainewasset.png)

3. Give the agent the name:
```
school_research_assistant
```

Select a model — **ibm/granite-3-3-8b-instruct** or **meta-llama/llama-3-3-70b-instruct** both work well for this use case.

![alt text](../Screenshots/wxaiagentname.png)

4. In the **System instructions** field enter:

```
You are a NYSDOE School Research Assistant. When given a research query:
1. Use the Google Search tool to find relevant, current information.
2. Filter results for New York State education context where possible.
3. Return a concise, structured summary (3–5 bullet points) with source URLs.
Focus areas: emergency protocols, shelter locations, district policies, school closures.
```

![alt text](../Screenshots/wxaisystemprompt.png)

5. In the **Tools** panel, click **Add tool** and select **Google Search** from the list of available tools:

![alt text](../Screenshots/wxaiaddgooglesearch.png)

> 💡 If Google Search does not appear in the tools list, ask your instructor — it may need to be enabled at the project level first.

6. Click **Preview** and send the following test query to validate the agent:

```
What are the current NYSDOE emergency weather protocols for school districts?
```

✅ You should see the agent call the Google Search tool and return a structured summary with bullet points and source URLs.

![alt text](../Screenshots/wxaiagenttest.png)

---

### Step 2: Deploy the Agent in watsonx.ai

1. Click **Save**, then click **Deploy** to publish the agent as a hosted endpoint.

2. Select a deployment space (your instructor will tell you which space to use), give the deployment a name, and click **Deploy**:

![alt text](../Screenshots/wxaideploy.png)

3. Once deployment is complete, open the deployment details and copy the **Endpoint URL** — you will need it in the next step.

4. You will also need an **API key**. In the top-right of the IBM Cloud / watsonx.ai interface, go to **Manage → API Keys** and create a new key. Copy and save it.

![alt text](../Screenshots/wxaiendpointurl.png)

---

### Step 3: Import the watsonx.ai Agent into Watson Orchestrate

1. Switch back to the Watson Orchestrate browser tab. In the **Agent Builder**, click **Add External Agent**:

![alt text](../Screenshots/addexternalagent.png)

2. Fill in the details using the endpoint and key from the previous step:

   | Field | Value |
   |---|---|
   | **Name** | `school_research_assistant` |
   | **Title** | `School Research Assistant` |
   | **Endpoint URL** | *(the URL you copied from the watsonx.ai deployment)* |
   | **Authentication** | API Key |
   | **API Key** | *(the API key you created)* |

![alt text](../Screenshots/externalagentconfig.png)

3. Click **Test Connection**. You should see a green success indicator:

![alt text](../Screenshots/connectiontestsuccess.png)

4. Click **Save**.

---

### Step 4: Add the Research Assistant to the Orchestrator

1. Open the **orchestrator_agent** in the Agent Builder.

2. Update the **Behavior** (Instructions) to add a third routing rule:

```
3. RESEARCH OR POLICY LOOKUP (questions about district protocols, shelter
   availability, NYSDOE policies, contextual background)
   → Route to: school_research_assistant
   Pass: a precise research query derived from the user's request.
```

3. Scroll down to the **Agents** section and click **Add Agent**. Search for and add `school_research_assistant`:

![alt text](../Screenshots/addresearchassistant.png)

4. Click **Save**.

5. Test the updated Orchestrator by sending:

```
What are the current NYSDOE emergency weather protocols for school districts?
```

✅ The Orchestrator should route this to the School Research Assistant and return a structured summary with source URLs.

![alt text](../Screenshots/researchoutput.png)

---

## Part 5 — End-to-End Testing

Now test all scenarios end-to-end through the **Orchestrator Agent** preview.

---

### Test 1 — Use Case 1: Facility Relocation

1. Send the following to the Orchestrator:

```
A water main broke at M007 in District 4. Building code M007.
We need to relocate immediately and need 8 rooms starting 2025-07-15.
```

![alt text](../Screenshots/test1message.png)

✅ **Expected flow:** Orchestrator → Operations Agent → `relocation_flow` → booking confirmation + relocation plan + parent email + staff email

![alt text](../Screenshots/test1output.png)

---

### Test 2 — Use Case 2: Emergency Threat Response

2. Send the following to the Orchestrator:

```
A tornado watch has been issued for District 7. Severity is HIGH.
Take all necessary emergency actions across all buildings in the district.
```

![alt text](../Screenshots/test2message.png)

✅ **Expected flow:** Orchestrator → Emergency Response Agent → `threat_response_flow` → incident report showing HVAC and door status values

![alt text](../Screenshots/test2output.png)

---

### Test 3 — Combined: Research + Emergency Response *(Optional — requires Part 4)*

3. Send the following to the Orchestrator:

```
A severe ice storm warning has been issued for District 7. Before executing
emergency protocols, look up the current NYSDOE winter storm school closure
policy, then act accordingly.
```

![alt text](../Screenshots/test3message.png)

✅ **Expected flow:** Orchestrator → School Research Assistant (policy lookup) → Emergency Response Agent (execute protocols) → consolidated report

![alt text](../Screenshots/test3output.png)

---

## Pulling it all together

In this lab we built two agentic workflows entirely in the Watson Orchestrate UI and wired them to specialist agents under a central Orchestrator.

**Agentic workflows** gave us deterministic, multi-step execution — every building is checked, every principal is alerted, every room is booked in the correct order. This is critical for real emergency operations where steps cannot be skipped or reordered.

**Prompt nodes** inside the flows used actual API output values (building names, booking IDs, alert counts, HVAC status) to generate professional documents — booking confirmations, relocation plans, parent emails, and incident reports — with no hallucinated placeholders.

**Specialist agents** stay focused on a single domain (operations vs. emergency) while the **Orchestrator** handles routing and consolidation. Adding a new use case in the future means building one new flow and one new agent — no changes to the existing components.

**External agents** extend Watson Orchestrate with capabilities hosted outside the platform. The School Research Assistant runs on watsonx.ai but integrates seamlessly as a collaborator to the Orchestrator.

With this architecture, a district coordinator can resolve a facility emergency or weather crisis in seconds instead of hours — and the entire process is auditable, repeatable, and consistent every time.
