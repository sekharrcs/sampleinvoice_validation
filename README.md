flowchart TD
    A[Client / Caller] -->|HTTP POST /orchestrators/process_invoice| B[Durable Functions App<br/>invoice-processor-durable/function_app.py]
    B -->|Start orchestration| C[invoice_processing_orchestrator<br/>function_app.py]
    C -->|call_activity: process_invoice_activity| D[Activity<br/>activities/invoice_activities.py]
    D -->|InvoiceAgent.create_and_process| E[Agent Orchestrator<br/>agents/invoice_agent.py]

    E -->|Tool call: extract_invoice_text| F[Tool Function<br/>agents/invoice_functions.py]
    F -->|HTTP POST /extract + PDF| G[PDF Extractor API<br/>pdf-extractor-api/main.py]
    G -->|extract_text_from_pdf| H[pdf_utils.py]
    G -->|analyze_page via Azure OpenAI| I[ai_utils.py]

    I -->|JSON per page| G
    G -->|JSON response| F
    F -->|Raw extraction JSON| E
    E -->|Validated response schema (AgentResponse)| D
    D -->|Persist JSON| J[processed_requests/*.json]
    D -->|Return result| C
    C -->|Durable result/status| A
