Training User Guide - Workday Generate Test Data

Access GenWizard, trigger a persona, create a persona, configure a tool, and run a backend agent for: Generate Test Data - GTD
Version 2.0  |  19 August 2026
 
Index
1. Access GenWizard
1.1 Open the GenWizard link
1.2 Sign in to GenWizard
2. Select and trigger the persona
2.1 View and select a persona
2.2 Start the conversation
2.3 Download the generated output
3. Other Options of GTD
3.1 Assign Roles
3.2 Job Posting
3.3 Position Data
3.4 Candidate Data
3.5 Hire Data
4. Create a persona
4.1 Open Personas
4.2 Create a new persona
4.3 Enter persona details
4.4 Configure conversation starters and datasource
4.5 Connect a tool and configure attachments
4.6 Save or update the persona
5. Create a tool
5.1 Open Tool Orchestrator
5.2 Create a new tool
5.3 Enter tool details
5.4 Configure the Run function
5.5 Update the Agent ID
5.6 Update the API URL and API Key
5.7 Certify the code and add attributes
5.8 Save the tool
6. Add the tool to the persona
7. Run the configured persona
8. GenWizard Access Issues
1. Access GenWizard
1.1 Open the GenWizard link
The GenWizard link varies depending on the instance being used. Access the link for the required GenWizard instance.
The standard URL for the development instance is: https://platforms-dev.genwizard.accenture.com/chatbot/chat
1. Open the GenWizard link for the required instance in a supported browser.
 
Figure 1. Open the link for the required GenWizard instance.
1.2 Sign in to GenWizard
1. On the sign-in page, enter the email address associated with your account.
2. Select Next.
3. Complete the authentication process using the assigned sign-in method.
 
Figure 2. Enter the account details and continue the sign-in process.
4. After authentication is complete, the GenWizard chatbot home page is displayed.
 
Figure 3. GenWizard chatbot home page.
2. Select and Trigger an Existing Persona
2.1 View and select a persona
1. Select or enter @ in the chatbot input.
2. A list of available personas is displayed. Find the Workday QA Agent persona by its name and select it.
 
Figure 4. Enter @ and select the Workday QA Agent persona by name.
3. Confirm that the selected persona name appears beside the @ symbol in the chatbot input.
4. Type Run or Trigger to start the selected persona
 
Figure 5. Confirm the selected persona and enter a message.
2.2 Start the conversation
1. Enter a message to start the conversation and select the arrow button or enter to submit it.
2. Review the persona response. It is asking for a user input for which agent it should run for. Select option 2: Generate Test Data  
Figure 6. Review the persona response and select the option which is required

4. The persona would give user seven options from which the user has to select, in this case option A: Test Data generation for Job Requisition is selected
 
Figure 7. Review the persona response and select the option which is required


5. After selecting the option, the persona asks the user to upload the input excel file:
I.	Select the Attach icon in the chatbot input.
 
Figure 8. Select the attachment icon
II.	In Add context to Assist GenAI, select Upload Files

 
Figure 9. Open Upload Files and add the required file.
III.	Select Drop files or drag and drop the required file into the upload area.
IV.	Confirm that the uploaded file is displayed.
V.	Select Add context.
 
Figure 10. Confirm the uploaded file and select Add context.


6. After the required attachment has been added, enter trigger in the chatbot input and submit the request.
7. Upon submitting the request, persona asks to select the countries from the list of countries it provides. User can select up to 3 countries. Here we are giving the India & USA
 
 
Figure 11. Review the persona response and select the options which is required

8. After entering countries as inputs, persona asks for the number of Job Requisitions to be created. Here we are giving 10. 
Figure 12. Review the persona response and give the desired input
9. After entering number of Job Requisitions, persona asks for Job Requesition ID Format  
Figure 13. Review the persona response and give the desired input format
10. After inputting the Job Requisition ID Format, persona asks for Recruiting Start Date in YYYY-MM-DD format
 
Figure 14. Review the persona response and give the desired input format

11. After the tool has triggered the backend agent, the status changes to Done and a confirmation message is displayed.
 
Figure 15. The status changes to Done after the backend agent is triggered.
2.3 Download the generated output
1. After backend processing is complete, review the completion message.
2. If the output is a document or another downloadable file, select the displayed download link.
 
Figure 16. Select the link provided for the generated output file.
3. The Agent Manager download page opens and the browser begins downloading the generated file.
 
Figure 17. The Agent Manager page confirms that the artifact is being downloaded.
After selecting the download link, open the downloaded report. Here is the output for the above generated report. We are only mentioning the the main part of the report to show the functionality of the agent
 
Figure 18. Example downloaded Closed Cases report with synthetic data.
3. Other Steps of GTD
3.1 Creation of test data for Assign Roles
1. Assign Roles also has a very similar execution to Job Requisition.
2. The attached images can be used as a reference for the persona’s execution



I. Persona Inputs
 
 
 
 
 
 
 

II. Agent Trigger
 


III. Output Example
 
3.2 Creation of test data for Job Posting
1. Jop Posting also has a very similar execution to Job Requisition.
2. The attached images can be used as a reference for the persona’s execution
3. There is one additional option as input, which is: Job Posting Site
I. Persona Inputs
 
 
 
 
 

 
 
 
The only additional option is that it asks the user in which platform the Job Posting should be posted.

II. The Agent triggers
 




III.  Output Example
 

3.3 Creation of test data for Position Data
1. Position Data also has a very similar execution to Job Requisition.
2. The attached images can be used as a reference for the persona’s execution
I. Persona Inputs
 
 
 
 
 
 
 
 

II. The Agent Triggers
 

III. Output Example
 

3.4 Creation of test data for Candidate Data
1. Jop Posting also has a very similar execution to Job Requisition.
2. The attached images can be used as a reference for the persona’s execution
3. There is one additional option as input, which is: Candidate Stage

I. Persona Inputs


 
 
 
 
 
 
 
 
 
The only additional option is that it asks the user the Candidate Stage
II. Agent Execution
 

3.5 Creation of test data for Hire Data
1. Jop Posting also has a very similar execution to Job Requisition.
2. The attached images can be used as a reference for the persona’s execution
3. There is one additional option for input, which is: Hire Reason

I. Persona Inputs
 
 
 
 
 
 
 
The only additional option is that it asks the user the Hire Reason
 
II. Agent Execution
 
4. Create a Persona
4.1 Open Personas
1. Select the Settings icon in the upper-right corner.
2. Select Personas.
 
Figure 19. Open Settings and select Personas.
4.2 Create a new persona
1. On the Your Personas page, select + New Persona.
 
Figure 20. Select New Persona.
4.3 Enter persona details
1. Enter a clear name in Name. The name is displayed when users select @ in the chatbot.
2. Enter a description explaining the purpose of the persona.
3. In Instructions, enter the prompt that defines how the agent should behave and respond.
 
Figure 21. Enter the persona name, description, and instructions.
4.4 Configure conversation starters and datasource
1. In Conversation starters, add one or more starting messages for the relevant scenarios.
2. If the persona needs an external datasource, select the required datasource from the Datasource drop-down list.
 
Figure 22. Configure conversation starters and select a datasource when required.
4.5 Connect a tool and configure attachments
1. In Tools, select the required tool. If the required tool has not yet been created, add it after completing the tool-creation process.
2. For Is Attachment Required, select Yes when users must upload a file before triggering the persona. Otherwise, select No.
 
Figure 23. Select the required tool and configure the attachment requirement.
4.6 Save or update the persona
1. Review the completed configuration and select Save.
 
Figure 24. Save the completed persona.
To update the persona later, open Settings, select Personas, search for the required persona, open it, update the required fields, and select Save.
5. Create a Tool
5.1 Open Tool Orchestrator
1. Select the Settings icon in the upper-right corner.
2. Select Tool Orchestrator.
 

Figure 25. Open Settings and select Tool Orchestrator.
5.2 Create a new tool
1. On the Your Tools page, select + New Tool.
 
Figure 26. Select New Tool.
5.3 Enter tool details
1. Enter the tool name and description.
2. For Tool Enabled, select Yes when the tool should be available for use.
3. If the tool triggers an external network, select Yes for the external network setting. Otherwise, select No.
 
Figure 27. Enter the tool details and configure Tool Enabled.
5.4 Configure the Run function
The Run function contains the Python source code that defines how the tool behaves. Use approved source code or the standard source template below.
 
Figure 28. Enter the Python source code in the Run function.
Standard Run Function Source Template
import httpx
from app.dependencies import get_config_service
from app.utils.dependencies import inject
import json
 
# Configuration for JWT token retrieval
def geturl():
    return "<GENWIZARD_BASE_URL>"
BASE_URL = geturl()
 
ATTACHMENTS = user_query_context.attachments
CONVERSATION_ID = user_query_context.conversation_id
JWT_TOKEN = user_query_context.user_specific_jwt_token
 
def getapiurl():
    return "<AGENT_MANAGER_GRAPHQL_URL>"
API_URL = getapiurl()
 
def getapikey():
    return "<AGENT_MANAGER_API_KEY>"
API_KEY = getapikey()
 
def get_agentid():
    return "<BACKEND_AGENT_ID>"
RUN_AGENT_TEAM_ID = get_agentid()
 
configurations = {}
TASK = {
    "conversationid": CONVERSATION_ID,
    "configurations": configurations,
    "filename": [],
    "requester_user_email_id": user_query_context.user_id,
    "request_env": "<REQUEST_ENVIRONMENT>"
}
 
async def get_jwt_token():
    headers = {"accept": "*/*", "Content-Type": "application/json"}
    params = {"useDeflate": "true"}
    data = {"username": "<USERNAME>", "password": "<PASSWORD>"}
    async with httpx.AsyncClient(verify=False) as client:
        response = await client.post(
            f"{BASE_URL}/atr-gateway/identity-management/api/v1/auth/token",
            headers=headers, params=params, json=data,
        )
        response.raise_for_status()
        return response.json()["token"]
 
async def get_attachment_content(attachment_id, headers):
    async with httpx.AsyncClient(verify=False) as client:
        response = await client.get(
            f"{BASE_URL}/atr-gateway/genai/attachments/{attachment_id}/content",
            headers=headers,
        )
        response.raise_for_status()
        return response.content
 
async def get_presigned_url(filename):
    async with httpx.AsyncClient() as client:
        payload = {
            "query": """mutation SetUploadedArtifact(
                $uploadedArtifact: UploadedArtifactInput!,
                $setUploadedArtifactId: String
            ) {
                setUploadedArtifact(
                    uploadedArtifact: $uploadedArtifact
                    id: $setUploadedArtifactId
                ) {
                    uploadedArtifact {
                        id name description createdBy createdOn
                        updatedBy updatedOn s3Key tags __typename
                    }
                    message presignedUrl __typename
                }
            }""",
            "variables": {
                "uploadedArtifact": {
                    "name": filename,
                    "description": filename
                }
            },
        }
        headers = {"Content-Type": "application/json", "x-api-key": API_KEY}
        response = await client.post(API_URL, json=payload, headers=headers)
        response.raise_for_status()
        data = response.json()
        return data["data"]["setUploadedArtifact"]["presignedUrl"]
 
async def upload_file(presigned_url, file_content, content_type):
    async with httpx.AsyncClient() as client:
        headers = {"Content-Type": content_type, "x-api-key": API_KEY}
        try:
            response = await client.put(
                presigned_url, headers=headers, content=file_content
            )
        except Exception:
            return None
        response.raise_for_status()
        return response.status_code
 
async def trigger_execution():
    async with httpx.AsyncClient() as client:
        payload = {
            "query": """mutation RunAgent(
                $runAgentId: String!,
                $type: RunAgentType!,
                $task: String!
            ) {
                runAgent(id: $runAgentId, type: $type, task: $task) {
                    message traceId __typename
                }
            }""",
            "variables": {
                "runAgentId": RUN_AGENT_TEAM_ID,
                "type": "TEAM",
                "task": json.dumps(TASK)
            },
        }
        headers = {"Content-Type": "application/json", "x-api-key": API_KEY}
        try:
            response = await client.post(API_URL, json=payload, headers=headers)
            response.raise_for_status()
            trace_id = response.json()["data"]["runAgent"]["traceId"]
            return (
                "Triggered the backend agent successfully. "
                f"Trace ID: {trace_id}."
            )
        except Exception as e:
            return {"message": "Failed to trigger execution", "error": str(e)}
 
async def upload_excel_document(attachments, jwt_token):
    from datetime import datetime
    for attachment in attachments:
        uploaded_filename = (
            f"{datetime.now().strftime('%y%m%d%H%M%S')}_"
            f"{attachment.filename}"
        )
        TASK["filename"] = uploaded_filename
        presigned_url = await get_presigned_url(uploaded_filename)
        TASK["presigned_url"] = presigned_url
        if not presigned_url:
            return None, "Failed to get presigned URL for uploaded file."
        headers = {
            "accept": "application/json",
            "apiToken": jwt_token,
            "Content-Type": "application/json",
        }
        try:
            content = await get_attachment_content(
                attachment.id, headers=headers
            )
            status = await upload_file(
                presigned_url,
                content,
                "application/vnd.openxmlformats-officedocument."
                "spreadsheetml.sheet"
            )
            return status, "Excel file uploaded successfully."
        except Exception as e:
            return None, f"Error uploading Excel file: {e}"
    return None, "No Excel file found."
 
async def main(configurations):
    if len(ATTACHMENTS) > 0:
        excel_status, excel_message = await upload_excel_document(
            ATTACHMENTS, JWT_TOKEN
        )
        if not excel_status:
            return "No valid file was uploaded. Cannot proceed."
    execution_response = await trigger_execution()
    TASK["configurations"] = configurations
    return execution_response
 
try:
    result = await main(configurations)
    return result
except Exception:
    return "Failed to trigger analysis."
5.5 Update the Agent ID
In the source template, the value returned by get_agentid() is the value that must be changed for the required backend agent.
def get_agentid():
    return "<BACKEND_AGENT_ID>"
1. Open the required backend agent.
2. Locate the final part of the backend agent URL.
3. Copy the Agent ID from the final part of the URL.
4. Replace <BACKEND_AGENT_ID> in get_agentid() with the copied Agent ID.
 
Figure 29. Locate the Agent ID at the end of the backend agent URL.
5.6 Update the API URL and API Key
The API URL and API key change according to the GenWizard instance in which the agents are being set up. Update both values with the details for the required instance.
1. In getapiurl(), replace <AGENT_MANAGER_GRAPHQL_URL> with the Agent Manager GraphQL URL for the required instance.
2. In getapikey(), replace <AGENT_MANAGER_API_KEY> with the API key for the same instance.
def getapiurl():
    return "<AGENT_MANAGER_GRAPHQL_URL>"

def getapikey():
    return "<AGENT_MANAGER_API_KEY>"
5.7 Certify the code and add attributes
1. Review the Python source code and select the certification checkbox.
2. If the tool requires an attribute, select + Add Attribute.
 
Figure 30. Certify the Python code and add an attribute when required.
3. Enter the attribute Name, Required setting, and Type.
 
Figure 31. Enter the attribute name, required setting, and type.
4. Enter the attribute Description and Default Value.
 
Figure 32. Enter the attribute description and default value.
5.8 Save the tool
1. Review the completed tool configuration.
2. Select Save.
 
Figure 33. Save the completed tool.
6. Add the Tool to the Persona
Complete this section if the tool was not selected while the persona was being created.
1. Open Settings and select Personas.
2. Search for and open the required persona.
3. Open Tools and select the newly created tool.
 
Figure 34. Select the created tool in the persona configuration.
4. Select Save to update the persona.
7. Run the Configured Persona
1. Select the GenWizard logo to return to the chatbot home page.
2. Enter @ and select the configured persona by name.
3. Start the conversation.
4. If an attachment is required, select the Attach icon, upload the file, and select Add context.
5. Enter trigger and submit the request.
6. Monitor the status until it changes from In progress... to Done.
7. If the output is a document or another downloadable file, select the link provided in the completion message.
8. GenWizard Access Issues
For access-related issues, reach out to the following POCs:
First POC: s.pandurang.kaldhone@accenture.com
Second POC: ravissant.markenday@accenture.com
