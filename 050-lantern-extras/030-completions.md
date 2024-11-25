# LLM Completion Calls

## Calling LLM Completion API

```sql
SET lantern_extras.llm_token='xxxx'; -- this will be used as api_token if it is not passed via arguments
SELECT llm_completion(
    user_prompt => 'User input', -- User prompt to LLM model
    model => 'gpt-4o', -- Model for runtime to use (default: 'gpt-4o')
    system_prompt => 'Provide short summary for the given text', -- System prompt for LLM (default: '')
    base_url => 'https://api.openai.com', -- If you have custom LLM deployment provide the server url. (default: OpenAi API URL)
    api_token => '<llm_api_token>', -- API token for LLM server. (default: inferred from lantern_extras.llm_token GUC)
    azure_entra_token => '', -- If this is Azure deployment it supports Auth with entra token too
    runtime => 'openai' -- Runtime to use. (default: 'openai'). Use `SELECT get_available_runtimes()` for list
);
```
