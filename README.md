# Databricks-practice

To disable the Genie Code autofill and autocomplete suggestions in your Databricks workspace, you can turn it off directly through your user settings. [1]  
Individual User Settings (Disable Autocomplete) 
If you want to stop Genie from automatically suggesting code as you type, follow these steps: 

1. Click your username in the upper-right corner of the Databricks workspace. 
2. Select Settings from the dropdown menu. 
3. In the left sidebar, navigate to User &gt; Developer. 
4. Scroll down to the Code Editor section. 
5. Toggle OFF the following settings: 

	• Automatic Genie Code Autocomplete (or Automatic Assistant Autocomplete) 
	• Autocomplete as you type (Optional: turns off all standard IDE suggestions) [1, 2]  

Workspace Admin Settings (Disable Workspace-Wide) 
If you are a workspace administrator and want to deactivate Genie and other AI assistive features for all users in the workspace: 

1. Click your username in the top bar and select Settings. 
2. In the left pane under Workspace admin, click the Advanced tab. 
3. Locate the AI-powered AI assistive features section. 
4. Click the toggle to Off. [3]  

(Note: Depending on your Databricks deployment, you can also manage broader access by toggling off Enable partner-powered AI features under the Feature enablement tab in your admin settings.) [4]  
If you would like, let me know: 

• Whether you are trying to turn it off just for yourself or globally for your team 
• If Genie is breaking your syntax or just causing interface lag 

[1] https://community.databricks.com/t5/data-engineering/is-there-a-way-to-deactivate-genie-auto-corretion/td-p/160587
[2] https://community.databricks.com/t5/data-engineering/turn-off-ai-assistance-in-notebooks/td-p/149222
[3] https://community.databricks.com/t5/administration-architecture/how-to-get-rid-of-a-pesky-gen-ai-feature-in-the-editor/td-p/91908
[4] https://learn.microsoft.com/en-us/azure/databricks/databricks-ai/partner-powered

