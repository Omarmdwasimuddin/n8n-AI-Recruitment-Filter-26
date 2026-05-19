## n8n-AI-Recruitment-Filter

#### Webhook e value set koro--->HTTP Method: POST---> Path: telent-ai--->Test URL copy koro--->index.html e paste koro--->webhook er Listen for even test click koro--->index.html open koro browser e--->pdf file upload koro

#### Webhook er +sign click koro--->search & click: Extract from File--->Operation: Extract From PDF--->Input Binary Field: resume--->click: Execute step

#### Extract from File er +sign click koro---> search & click: open ai---> click: Message a model--->Model daw--->Role: System--->Prompt daw--->Add opton click koro--->Output Format e Json Object daw--->Execute step click koro.

#### supabase e table create koro--->Name daw: CVs
![](https://imgur.com/KZ7SeQp.png)

#### Message a model er +sign click koro--->search & click: supabase--->click: Create a row--->value set koro & Execute step click koro

#### Create a row er +sign click koro--->search & click: if--->condition daw 70% match hole email send hobe
#### if er true er +sign click koro--->searach & click: gmail--->Send a message clickd koro--->value set koro
