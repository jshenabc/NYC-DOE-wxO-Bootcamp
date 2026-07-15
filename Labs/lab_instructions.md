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
   | **Client ID** | `c14a0da2b3111279c05e959c43dfbaef` |
   | **Client Secret** | `7b5552e42e5adf2ca9e7405bb0e706bb` |
   | **Scope** | `location` |

![alt text](../Screenshots/fillincreds.png)

![alt text](../Screenshots/credential_type.png)

6. In the **Configure live connection** page, you can click **Paste draft configuration**. It will automatically populate the values you filled in for the draft configuration.

![alt text](../Screenshots/pastedraft.png)

7. Click **Finish**

![alt text](../Screenshots/finishconnect.png)

> ✅ You should see the connection status change to **Connected**. If it shows an error, double-check the Client ID and Secret — make sure there are no extra spaces.

THIS IS WHERE I LEFT OFF 7/14 4:30 PM
---

### Step B: Import the NYC Location API tool

This tool calls the real NYSDOE Location API and returns the list of schools (`schoolDBN`) and offices (`locationCode`) for a district.

1. In Watson Orchestrate, click **Tools** in the left navigation panel.

2. Click **Import tool**.

3. Click **Upload file** and select:
   ```
   tools/nyc_location_api.yaml
   ```

4. When prompted to select a connection, choose **location_api**.

5. Click **Import**.

6. Confirm the tool `getDistrictAdminDetails` appears in your tool list.

> 💡 **What this tool returns:**
> - `schools[].schoolDBN` — the building code for each school building in the district
> - `offices[].locationCode` — the building code for each office building in the district
>
> These are the real NYCDOE codes used by the Security and Facilities APIs to lock doors, control HVAC, and send principal alerts.

---

### Step C: Import the mock Security and Facilities API tools

The Security and Facilities APIs are mock services that simulate door locking, HVAC control, room booking, and principal alerts. They accept the same `schoolDBN` and `locationCode` values returned by the real Location API.

1. In Watson Orchestrate, click **Tools** → **Import tool**.

2. Upload the following files one at a time (no connection required for any of them):

   | File | Tools it provides |
   |---|---|
   | `tools/nysdoe_facilities_api.yaml` | `bookBuilding`, `bulkSetHVAC` |
   | `tools/nysdoe_security_api.yaml` | `setDoorState`, `bulkAlertPrincipals` |

3. After importing both files confirm the following tools appear in your tool list:

   - `getDistrictAdminDetails` *(Location API — real)*
   - `bookBuilding` *(Facilities API — mock)*
   - `bulkSetHVAC` *(Facilities API — mock)*
   - `setDoorState` *(Security API — mock)*
   - `bulkAlertPrincipals` *(Security API — mock)*

If any are missing, ask your instructor before proceeding.

---

## Part 1 — Building the Operations Workflow (Use Case 1)

> **Scenario:** A water main has broken at a middle school in District 4. The Operations workflow will automatically find a relocation building, book rooms, and draft parent and staff notification emails.

### The flow we are building:

![alt text](./lab-assets/part1/flow_overview.png)

We will now walk through creating the above workflow step by step.

---

### Step 1: Create the relocation_flow and configure inputs and outputs

Open the Agent Builder in Watson Orchestrate by clicking **Build → Agent Builder** in the main menu.

Click **Create Agent**, then select **Create from scratch**. We are going to create a temporary agent to build and test our flow in — we will create the real Operations Agent in Part 2.

Name the agent **Operations Flow Test** and click **Create**.

Scroll down to the **Toolset** section and click **Add Tool**:

![alt text](./lab-assets/part1/add_tool.png)

Select **Create an agentic workflow**:

![alt text](./lab-assets/part1/create_workflow.png)

The flow editor will open. Click the **pencil icon** next to the flow name in the top-left corner to edit the flow name and description:

![alt text](./lab-assets/part1/edit_flow_name.png)

Change the name to:
```
relocation_flow
```

Change the description to:
```
Finds available relocation buildings in a district, books rooms, and drafts parent and staff notification emails.
```

Click **Add output** to define the output variable for the flow:

![alt text](./lab-assets/part1/add_output.png)

Select **String** for the type, name the variable:
```
relocation_summary
```
and click **Add**.

Click **Save**. Your flow should now have a Start node and an End node, with `relocation_summary` configured as the output:

![alt text](./lab-assets/part1/flow_start.png)

You can verify the output was added by clicking on the **End** node:

![alt text](./lab-assets/part1/end_node_output.png)

---

### Step 2: Add the get_district_info tool node

Hover over the arrow connecting the Start node to the End node and click on the **+** sign:

![alt text](./lab-assets/part1/add_step.png)

Select **Tool**. Search for `getDistrictAdminDetails` and select it:

![alt text](./lab-assets/part1/select_tool.png)

Click on the new node, then click the **pencil icon** to rename it to:
```
get_district_info
```

Click on **Edit data mapping** to configure the inputs for this tool:

![alt text](./lab-assets/part1/edit_data_mapping.png)

Map the following inputs. For each, click the **expression icon** (`fx`) in the corresponding row and enter the expression:

| Input field | Expression |
|---|---|
| `districtCode` | `str(flow.input.district_id).zfill(2)` |
| `serviceAccountID` | `"api.ibmtest"` |
| `Components` | `"schools,offices"` |

> 💡 `zfill(2)` zero-pads the district ID so `"4"` becomes `"04"` and `"7"` becomes `"07"` — the format the API expects.

After setting the mappings, click on the **Start** node and select **Edit flow inputs**:

![alt text](./lab-assets/part1/edit_flow_inputs.png)

Add the following input variables. For each, click **Add variable**, select the type, enter the name and description, then click **Add**:

| Variable name | Type | Required | Description |
|---|---|---|---|
| `district_id` | String | Yes | Numeric district ID, e.g. `"4"` |
| `affected_building_name` | String | Yes | Name of the building being evacuated |
| `disruption_type` | String | Yes | e.g. `"water main break"` |
| `rooms_needed` | Integer | No | Default: `8` |
| `start_date` | String | Yes | Format: `YYYY-MM-DD` |

![alt text](./lab-assets/part1/flow_inputs_defined.png)

---

### Step 3: Add the book_rooms tool node

Hover over the arrow connecting `get_district_info` to the End node and click **+**. Select **Tool** and search for `bookBuilding`. Select it and rename the node to:
```
book_rooms
```

Click **Edit data mapping** and set the following input mappings:

| Input field | Expression |
|---|---|
| `buildingCode` | `flow['get_district_info'].output['schools'][0]['schoolDBN']` |
| `requester` | `"Operations Agent"` |
| `reason` | `flow.input.disruption_type + " at " + flow.input.affected_building_name` |
| `rooms_needed` | `flow.input.rooms_needed` |
| `start_date` | `flow.input.start_date` |

> 💡 `flow['get_district_info'].output['schools'][0]['schoolDBN']` picks the first school's building code (`schoolDBN`) returned by the real Location API. The agent will use this to book rooms in that school building.

![alt text](./lab-assets/part1/book_rooms_mapping.png)

---

### Step 4: Add the draft_output prompt node

Hover over the arrow connecting `book_rooms` to the End node and click **+**. Select **Generative prompt**:

![alt text](./lab-assets/part1/add_generative_prompt.png)

Click on the new node and rename it to:
```
draft_output
```

Click on **Edit prompt settings**:

![alt text](./lab-assets/part1/edit_prompt_settings.png)

Enter the following **System prompt**:
```
You are the NYSDOE Operations Agent. You produce professional, clear district 
communications and operational plans. Always fill every field with real values 
— never use placeholders.
```

Enter the following **User prompt**:
```
A facility disruption has occurred. Using the details below, produce the full relocation output.

DISRUPTION DETAILS:
- Affected building: {affected_building_name}
- District: {district_id}
- Disruption type: {disruption_type}
- Effective date: {start_date}

BOOKING CONFIRMATION (from API):
- Booking ID: {booking_id}
- Relocation building: {relocation_building_name}
- Relocation building address: {relocation_building_address}
- Rooms booked: {rooms_booked}

Produce the following four sections, clearly separated.
IMPORTANT: The affected building ({affected_building_name}) is being EVACUATED.
The relocation building ({relocation_building_name} at {relocation_building_address})
is the NEW TEMPORARY SITE. Never confuse the two.

## ✅ BOOKING CONFIRMED
A short confirmation (3–4 lines): state the affected building being evacuated, 
the new temporary site name and address, booking ID, rooms booked, and effective date.

## 📋 RELOCATION PLAN
A structured operational plan for the operations team. Include:
1. Immediate actions (Day 0 — {start_date})
2. Logistics (Day 1): room setup, IT transfer, student records, meal service
3. Communications: confirm emails sent, update district website
4. Responsible parties: assign each step to a role
5. Return plan trigger: define the condition to return to {affected_building_name}

## 📧 PARENT EMAIL
Subject: Important: School Relocation Notice — {affected_building_name}

A full professional parent notification email. Include what happened, the new 
temporary site name and full address, effective date, next steps, and contact info.

## 📧 STAFF EMAIL
Subject: STAFF NOTICE: Facility Relocation — {affected_building_name}

A full professional staff notification email. Include the disruption detail, 
new site name and address, booking confirmation ID, facilities notes, and contact info.
```

Now we need to add the **input variables** that the prompt references in the curly braces `{}`. Click on **Add variable** and create the following String variables. You can add test values to each so you can run the prompt and validate it directly in the editor:

| Variable name | Sample test value |
|---|---|
| `affected_building_name` | `PS 142 - District 4` |
| `district_id` | `4` |
| `disruption_type` | `water main break` |
| `start_date` | `2025-07-15` |
| `booking_id` | `BK-20250715-001` |
| `relocation_building_name` | `District 4 Community Center` |
| `relocation_building_address` | `500 Main Street, Brooklyn, NY` |
| `rooms_booked` | `8` |

After adding all variables, click **Generate response** to run the prompt on the test values and verify the output looks correct:

![alt text](./lab-assets/part1/prompt_test_output.png)

Once you are satisfied, close the prompt settings.

---

### Step 5: Map the prompt inputs

Click on the `draft_output` node and select **Edit data mapping**. We need to wire the outputs from earlier nodes in the flow to the prompt input variables we just created:

Click the **variable icon** in each row and select the corresponding source:

| Prompt variable | Map from |
|---|---|
| `affected_building_name` | Flow inputs → `affected_building_name` |
| `district_id` | Flow inputs → `district_id` |
| `disruption_type` | Flow inputs → `disruption_type` |
| `start_date` | Flow inputs → `start_date` |
| `booking_id` | `book_rooms` → `booking_id` |
| `relocation_building_name` | `book_rooms` → `building_name` |
| `relocation_building_address` | `find_buildings` → `output[0].address` |
| `rooms_booked` | `book_rooms` → `rooms_booked` |

![alt text](./lab-assets/part1/prompt_data_mapping.png)

---

### Step 6: Connect the output and save

Finally, click on the **End** node and connect the `relocation_summary` output to the output of the generative prompt node. Select **draft_output → value** as the source:

![alt text](./lab-assets/part1/map_output_var.png)

Click **Done** to close the flow editor. Your completed flow should look like this:

![alt text](./lab-assets/part1/completed_flow.png)

✅ The `relocation_flow` is now built. We will attach it to the Operations Agent in Part 2.

---

## Part 1B — Building the Emergency Response Workflow (Use Case 2)

> **Scenario:** A tornado watch has been issued for District 7. The Emergency Response workflow will automatically fetch all buildings in the district, bulk-alert school principals by SMS, switch office HVAC to low power, lock office doors, and produce a full incident report.

### The flow we are building:

![alt text](./lab-assets/part1b/flow_overview.png)

---

### Step 1: Create the threat_response_flow and configure inputs and outputs

In the **Agent Builder**, create another temporary test agent — name it **Emergency Flow Test** — then scroll down to **Toolset**, click **Add Tool**, and select **Create an agentic workflow**.

Click the **pencil icon** on the flow name and update it to:
```
threat_response_flow
```

Update the description to:
```
Finds district buildings, bulk-alerts school principals, secures office HVAC and doors, and drafts a full incident report.
```

Click **Add output**, select **String**, name the variable:
```
incident_summary
```
and click **Add**. Click **Save**.

---

### Step 2: Add the get_district_info tool node

Click on the **Start** node and select **Edit flow inputs**. Add the following input variables:

| Variable name | Type | Required | Description |
|---|---|---|---|
| `district_id` | String | Yes | Affected district, e.g. `"7"` |
| `threat_type` | String | Yes | e.g. `"tornado watch"` |
| `severity` | String | No | Default: `"high"` |
| `alert_message` | String | No | Optional custom SMS override text |

Hover over the arrow from Start to End and click **+**. Select **Tool**, search for `getDistrictAdminDetails`, select it, and rename the node to:
```
get_district_info
```

Click **Edit data mapping** and set:

| Input field | Expression |
|---|---|
| `districtCode` | `str(flow.input.district_id).zfill(2)` |
| `serviceAccountID` | `"api.ibmtest"` |
| `Components` | `"schools,offices"` |

> 💡 This calls the real NYC Location API. The response gives you `schools[].schoolDBN` (school building codes) and `offices[].locationCode` (office building codes). The next nodes use these to alert principals and secure the superintendent office.

![alt text](./lab-assets/part1b/find_buildings_mapping.png)

---

### Step 3: Add the alert_principals tool node

Hover over the arrow after `find_buildings` and click **+**. Select **Tool**, search for `bulkAlertPrincipals`, select it, and rename the node to:
```
alert_principals
```

Click **Edit data mapping** and set:

| Input field | Expression |
|---|---|
| `district_id` | `int(flow.input.district_id)` |
| `message` | `flow.input.alert_message if flow.input.alert_message else "[NYSDOE EMERGENCY — District " + flow.input.district_id + "] SEVERITY: " + flow.input.severity.upper() + ". Threat: " + flow.input.threat_type.upper() + ". Initiate shelter-in-place protocol immediately. Account for all students and staff. Await further instructions."` |
| `triggered_by` | `"Emergency Response Agent"` |
| `severity` | `flow.input.severity` |

![alt text](./lab-assets/part1b/alert_principals_mapping.png)

---

### Step 4: Add the set_office_hvac tool node

Hover over the arrow after `alert_principals` and click **+**. Select **Tool**, search for `bulkSetHVAC`, select it, and rename the node to:
```
set_office_hvac
```

Click **Edit data mapping** and set:

> 💡 `bulkSetHVAC` targets **all buildings in the district** by `districtCode` in a single API call — no need to look up individual office building codes. The response includes `buildings_count` so the prompt can report exactly how many offices were secured.

| Input field | Expression |
|---|---|
| `districtCode` | `str(flow.input.district_id).zfill(2)` |
| `mode` | `"low_power"` |
| `triggered_by` | `"Emergency Response Agent"` |
| `reason` | `flow.input.threat_type.title() + " — District " + flow.input.district_id + " (" + flow.input.severity + " severity)"` |

![alt text](./lab-assets/part1b/set_hvac_mapping.png)

---

### Step 5: Add the lock_office_doors tool node

Hover over the arrow after `set_office_hvac` and click **+**. Select **Tool**, search for `setDoorState`, select it, and rename the node to:
```
lock_office_doors
```

Click **Edit data mapping** and set:

| Input field | Expression |
|---|---|
| `building_id` | `next((b.building_id for b in flow['find_buildings'].output if b.building_type == 'office'), '')` |
| `action` | `"locked"` |
| `triggered_by` | `"Emergency Response Agent"` |
| `reason` | `flow.input.threat_type.title() + " — District " + flow.input.district_id + " (" + flow.input.severity + " severity)"` |
| `severity` | `flow.input.severity` |

![alt text](./lab-assets/part1b/lock_doors_mapping.png)

---

### Step 6: Add the draft_report prompt node

Hover over the arrow after `lock_office_doors` and click **+**. Select **Generative prompt** and rename the node to:
```
draft_report
```

Click **Edit prompt settings** and enter the following **System prompt**:
```
You are the NYSDOE Emergency Response Agent. Produce a concise, operational
incident report using only the values provided. Never use placeholders.
```

Enter the following **User prompt**:
```
A severe weather warning has triggered a district-wide threat response.
Using the values below, produce a complete incident report.

THREAT DETAILS:
- District: {district_id}
- District name: {district_name}
- Threat type: {threat_type}
- Severity: {severity}

LOCATION DATA (from real NYC Location API):
- Total schools in district (schoolDBN buildings): {school_count}
- Total office buildings HVAC-secured (all district offices): {hvac_buildings_count}

PRINCIPAL ALERT RESULTS (school buildings):
- Principals alerted: {alerted_count}
- Principals skipped (no phone on record): {skipped_count}

OFFICE ACTION RESULTS (all office buildings in district):
- Bulk HVAC status: {hvac_status} ({hvac_buildings_count} buildings set to low_power)
- Door lock status: {door_status}

Produce the following three sections, clearly separated.

## INCIDENT SUMMARY
Write a short operational summary for District {district_id} ({district_name})
describing the {threat_type} threat at severity {severity}. Confirm that principal
SMS alerts were sent to all {school_count} school buildings and that HVAC was bulk-set
to low_power across all {hvac_buildings_count} office buildings in the district,
and that office doors were locked.

## ACTION LOG
List every completed action as a bullet:
• bulkAlertPrincipals — {alerted_count} school principals alerted, {skipped_count} skipped
• bulkSetHVAC — {hvac_buildings_count} office buildings set to low_power — status: {hvac_status}
• setDoorState — superintendent office doors locked — status: {door_status}

## FOLLOW-UP
List immediate follow-up steps for the district coordinator.
If {skipped_count} > 0, note that those principals require manual outreach.
If {hvac_status} is not 'bulk_updated', flag the HVAC action for manual verification.
If {door_status} is not 'updated', flag the door lock for manual verification.
```

Add the following **String input variables** for the prompt. Add test values so you can validate the prompt without running the whole flow:

| Variable name | Sample test value |
|---|---|
| `district_id` | `7` |
| `district_name` | `COMMUNITY SCHOOL DISTRICT 07` |
| `threat_type` | `tornado watch` |
| `severity` | `high` |
| `school_count` | `41` |
| `hvac_buildings_count` | `13` |
| `alerted_count` | `41` |
| `skipped_count` | `0` |
| `hvac_status` | `bulk_updated` |
| `door_status` | `updated` |

Click **Generate response** to validate the prompt output looks like a proper incident report, then close the prompt settings:

![alt text](./lab-assets/part1b/prompt_test_output.png)

---

### Step 7: Map the prompt inputs and connect the output

Click on the `draft_report` node and select **Edit data mapping**. Map each variable to the correct source:

| Prompt variable | Map from | Expression (fx) |
|---|---|---|
| `district_id` | Flow inputs | `str(flow.input.district_id)` |
| `district_name` | `get_district_info` | `flow['get_district_info'].output['districtName']` |
| `threat_type` | Flow inputs | `flow.input.threat_type` |
| `severity` | Flow inputs | `flow.input.severity` |
| `school_count` | `get_district_info` | `len(flow['get_district_info'].output['schools'])` |
| `hvac_buildings_count` | `set_office_hvac` | `flow['set_office_hvac'].output['buildings_count']` |
| `alerted_count` | `alert_principals` | `len(flow['alert_principals'].output.alerted)` |
| `skipped_count` | `alert_principals` | `len(flow['alert_principals'].output.skipped)` |
| `hvac_status` | `set_office_hvac` | `flow['set_office_hvac'].output['status']` |
| `door_status` | `lock_office_doors` | `flow['lock_office_doors'].output.status` |

![alt text](./lab-assets/part1b/prompt_data_mapping.png)

Click on the **End** node and connect `incident_summary` to **draft_report → value**.

Click **Done** to close the flow editor. Your completed flow should look like this:

![alt text](./lab-assets/part1b/completed_flow.png)

✅ The `threat_response_flow` is now built.

---

## Part 2 — Building Agents and Wiring Everything Together

Now that both flows are built, we will create the real specialist agents, connect an external research agent, and build the Orchestrator that routes everything together.

---

### Create the Operations Agent

Open the **Agent Builder** and click **Create Agent**. Select **Create from scratch**.

Name the agent:
```
operations_agent
```

Give it the following description:
```
Handles NYSDOE district-level facility disruptions. Given a disruption trigger (water main break, power outage, building closure), this agent finds available relocation buildings, books rooms, and drafts parent and staff notification emails.
```

![alt text](./lab-assets/part2/create_operations_agent.png)

Select **meta-llama/llama-3-3-70b-instruct** from the **Model** drop-down.

Scroll down to the **Behavior** (Instructions) section and enter:
```
You are the NYSDOE Operations Agent. When a facility disruption is reported,
invoke the relocation_flow tool immediately — do not ask questions first.

Extract these inputs from the message:
- district_id, affected_building_name, disruption_type, rooms_needed (default 8), start_date

If district_id cannot be determined, ask for it once before invoking the flow.

Present the full flow output to the user (booking confirmation, relocation plan,
parent email, staff email), then ask:
"Would you like to send these notifications? Reply YES to confirm or NO to discard."

If no suitable building is found, report it clearly and suggest escalating to the Orchestrator.
```

![alt text](./lab-assets/part2/operations_agent_instructions.png)

Scroll down to the **Toolset** section and click **Add Tool**. This time, instead of creating a new workflow, select **Add existing** and search for `relocation_flow`. Select it and add it:

![alt text](./lab-assets/part2/add_relocation_flow.png)

**Test the Operations Agent** by clicking **Preview** and sending:
```
A water main broke at PS 142 in District 4. We need to relocate immediately.
The disruption started today and we need 8 rooms starting 2025-07-15.
```

✅ The agent should call `relocation_flow` and return a booking confirmation, relocation plan, and two draft emails.

![alt text](./lab-assets/part2/operations_agent_test.png)

Click **Save**.

---

### Create the Emergency Response Agent

Click **Create Agent** → **Create from scratch**.

Name the agent:
```
emergency_response_agent
```

Description:
```
District-Wide Threat Response Agent. Triggered by a severe weather warning for a specific geographic district. Differentiates response by building type: for schools it bulk-alerts every principal by SMS; for office buildings it switches HVAC to low-power mode and remotely locks all doors. Returns a full timestamped incident report.
```

Select **meta-llama/llama-3-3-70b-instruct** from the **Model** drop-down.

Enter the following **Behavior** (Instructions):
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

![alt text](./lab-assets/part2/emergency_agent_instructions.png)

Under **Toolset**, click **Add Tool** → **Add existing** → search for `threat_response_flow` and add it:

![alt text](./lab-assets/part2/add_threat_flow.png)

**Test the Emergency Response Agent** by clicking **Preview** and sending:
```
A tornado watch has been issued for District 7. Severity is HIGH.
Take all necessary emergency actions across all buildings in the district.
```

✅ The agent should return an incident report with an INCIDENT SUMMARY, ACTION LOG showing the new HVAC and door status values, and a FOLLOW-UP section.

![alt text](./lab-assets/part2/emergency_agent_test.png)

Click **Save**.

---

### Build and Connect the School Research Assistant (watsonx.ai)

In this section you will build the **School Research Assistant** directly inside **watsonx.ai** as a LangGraph agent, then connect it to Watson Orchestrate as an external agent so the Orchestrator can call it.

---

#### Step 1: Open watsonx.ai and create a new AI Agent

Open a new browser tab and navigate to your watsonx.ai instance. In the left navigation click **Projects**, then open the lab project provided by your instructor.

Inside the project click **New asset** → **AI Agent**:

![alt text](./lab-assets/part2/wxai_new_asset.png)

---

#### Step 2: Configure the agent name and model

Give the agent the name:
```
school_research_assistant
```

Select a model — **ibm/granite-3-3-8b-instruct** or **meta-llama/llama-3-3-70b-instruct** both work well for this use case.

![alt text](./lab-assets/part2/wxai_agent_name_model.png)

---

#### Step 3: Add the system prompt

In the **System instructions** field enter:

```
You are a NYSDOE School Research Assistant. When given a research query:
1. Use the Google Search tool to find relevant, current information.
2. Filter results for New York State education context where possible.
3. Return a concise, structured summary (3–5 bullet points) with source URLs.
Focus areas: emergency protocols, shelter locations, district policies, school closures.
```

![alt text](./lab-assets/part2/wxai_system_prompt.png)

---

#### Step 4: Add the Google Search tool

In the **Tools** panel, click **Add tool** and select **Google Search** from the list of available tools:

![alt text](./lab-assets/part2/wxai_add_google_search.png)

> 💡 If Google Search does not appear in the tools list, ask your instructor — it may need to be enabled at the project level first.

---

#### Step 5: Test the agent in watsonx.ai

Click **Preview** and send the following test query:

```
What are the current NYSDOE emergency weather protocols for school districts?
```

✅ You should see the agent call the Google Search tool and return a structured summary with bullet points and source URLs.

![alt text](./lab-assets/part2/wxai_agent_test.png)

---

#### Step 6: Deploy the agent

Click **Save**, then click **Deploy** to publish the agent as a hosted endpoint.

Select a deployment space (your instructor will tell you which space to use), give the deployment a name, and click **Deploy**:

![alt text](./lab-assets/part2/wxai_deploy.png)

Once deployment is complete, open the deployment details and copy the **Endpoint URL** — you will need it in the next step.

You will also need an **API key**. In the top-right of the IBM Cloud / watsonx.ai interface, go to **Manage → API Keys** and create a new key. Copy and save it.

![alt text](./lab-assets/part2/wxai_endpoint_url.png)

---

#### Step 7: Connect the deployed agent to Watson Orchestrate

Switch back to the Watson Orchestrate browser tab. In the **Agent Builder**, click **Add External Agent**:

![alt text](./lab-assets/part2/add_external_agent.png)

Fill in the details using the endpoint and key from the previous step:
- **Name:** `school_research_assistant`
- **Title:** `School Research Assistant`
- **Endpoint URL:** *(the URL you copied from the watsonx.ai deployment)*
- **Authentication:** API Key
- **API Key:** *(the API key you created)*

![alt text](./lab-assets/part2/external_agent_config.png)

Click **Test Connection**. You should see a green success indicator:

![alt text](./lab-assets/part2/connection_test_success.png)

**Test the connection** by opening the agent preview in Orchestrate and sending:
```
What are the current NYSDOE emergency weather protocols for school districts?
```

✅ You should receive the same structured summary you saw in watsonx.ai, now routed through the Orchestrate connection.

Click **Save**.

---

### Create the Orchestrator Agent

Click **Create Agent** → **Create from scratch**.

Name the agent:
```
orchestrator_agent
```

Description:
```
Central NYSDOE Orchestrator Agent. Triages every incoming district trigger and routes it to the correct specialist: facility disruptions go to the Operations Agent, weather or security threats go to the Emergency Response Agent, and research or policy questions go to the School Research Assistant.
```

Select **meta-llama/llama-3-3-70b-instruct** from the **Model** drop-down.

Enter the following **Behavior** (Instructions):
```
You are the NYSDOE Orchestrator Agent. Your sole job is to triage incoming
triggers and delegate to the appropriate specialist agent. Never attempt to
handle facility operations or emergency protocols yourself.

ROUTING RULES:

1. FACILITY DISRUPTION (water main break, power outage, building closure,
   structural damage, HVAC failure, relocation needed)
   → Route to: operations_agent
   Pass: district_id, affected building name, disruption type, rooms needed, start date.

2. WEATHER OR SECURITY THREAT (tornado watch, ice storm, hurricane, severe
   weather warning, lockdown)
   → Route to: emergency_response_agent
   Pass: district_id, threat_type, and severity.

3. RESEARCH OR POLICY LOOKUP (questions about district protocols, shelter
   availability, NYSDOE policies, contextual background)
   → Route to: school_research_assistant
   Pass: a precise research query derived from the user's request.

4. COMBINED SCENARIO (research needed before taking action)
   → First route to: school_research_assistant
   → Then route to: operations_agent or emergency_response_agent
   → Consolidate both outputs before responding.

BEHAVIOUR:
- Extract all context from the message before routing.
- If district_id is missing, ask for it once — then route immediately.
- Do not ask unnecessary clarifying questions.
- Present the specialist agent's full output without summarising or truncating.
- If a specialist agent fails, report the failure clearly.
```

![alt text](./lab-assets/part2/orchestrator_instructions.png)

Scroll down to the **Agents** section (separate from Toolset). Click **Add Agent** and add all three specialist agents:
- `operations_agent`
- `emergency_response_agent`
- `school_research_assistant`

![alt text](./lab-assets/part2/add_collaborators.png)

Click **Save**.

---

## End-to-End Testing

Now test all three scenarios through the **Orchestrator Agent** preview.

### Test 1 — Use Case 1: Facility Relocation

Send the following to the Orchestrator:
```
A water main has burst at PS 142 in District 4. The building must be evacuated 
and students relocated. We need 8 rooms starting 2025-07-15. Handle this.
```

✅ **Expected flow:** Orchestrator → Operations Agent → `relocation_flow` → booking confirmation + relocation plan + parent email + staff email

![alt text](./lab-assets/part3/test1_output.png)

---

### Test 2 — Use Case 2: Emergency Threat Response

```
A tornado watch has been issued for District 7. Severity is HIGH.
Take all necessary emergency actions across all buildings in the district.
```

✅ **Expected flow:** Orchestrator → Emergency Response Agent → `threat_response_flow` → incident report showing new HVAC and door status values

![alt text](./lab-assets/part3/test2_output.png)

---

### Test 3 — Combined: Research + Emergency Response

```
A severe ice storm warning has been issued for District 12. Before executing 
emergency protocols, look up the current NYSDOE winter storm school closure 
policy, then act accordingly.
```

✅ **Expected flow:** Orchestrator → School Research Assistant (policy lookup) → Emergency Response Agent (execute protocols) → consolidated report

![alt text](./lab-assets/part3/test3_output.png)

---

## Pulling it all together

In this lab we built two agentic workflows entirely in the Watson Orchestrate UI and wired them to specialist agents under a central Orchestrator.

**Agentic workflows** gave us deterministic, multi-step execution — every building is checked, every principal is alerted, every room is booked in the correct order. This is critical for real emergency operations where steps cannot be skipped or reordered.

**Prompt nodes** inside the flows used actual API output values (building names, booking IDs, alert counts, HVAC status) to generate professional documents — booking confirmations, relocation plans, parent emails, and incident reports — with no hallucinated placeholders.

**Specialist agents** stay focused on a single domain (operations vs. emergency) while the **Orchestrator** handles routing and consolidation. Adding a new use case in the future means building one new flow and one new agent — no changes to the existing components.

**External agents** extend Watson Orchestrate with capabilities hosted outside the platform. The School Research Assistant runs on watsonx.ai but integrates seamlessly as a collaborator to the Orchestrator.

With this architecture, a district coordinator can resolve a facility emergency or weather crisis in seconds instead of hours — and the entire process is auditable, repeatable, and consistent every time.
