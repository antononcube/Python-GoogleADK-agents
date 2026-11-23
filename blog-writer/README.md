# Blog writer AI-agent

The AI-agent "blog-writer" was created using several iterations of code generation 
using OpenAI's Codex AI-agent -- see the documents ["prompts.md"](./docs/prompts.md).

The initial, basic structure of the agent is this one:

```mermaid
flowchart TD
    Start(('Start'))
    End((End))
    Writer["Generate or edit content"]
    CodeReviewer
    ContentReviewer
    CopyEditor
    Judge

    Start --> Writer
    Writer --> CodeReviewer
    Writer --> ContentReviewer
    Writer --> CopyEditor
    CodeReviewer --> Judge
    ContentReviewer --> Judge
    CopyEditor --> Judge
    Judge --> |needs work| Writer
    Judge --> |good| End
```

But that structure does not allow for reliable handling of meta queries for process status,
export of drafts or results, or self-knowledge / introspection. 
Hence, agent's underlying graph was extended to have an orchestrator and exporter nodes.

Here is the current graph: 

```mermaid

flowchart TB
    BW["✦ blog_writer"]

    CW["✦ content_workflow"]
    EX["✦ exporter"]
    GWS["🔧 get_workflow_status"]
    GEI["🔧 get_export_info"]
    CSA["🔧 check_source_access"]

    SR["✦ source_researcher"]
    RL["✦ revision_loop"]
    ED["🔧 export_draft"]
    GC["🔧 get_content"]

    WRJ["✦ write_review_judge"]
    W["✦ writer"]
    R["✦ reviewers"]
    J["✦ judge"]

    CR["✦ code_reviewer"]
    CTR["✦ content_reviewer"]
    CE["✦ copy_editor"]
    AD["🔧 approve_draft"]

    BW --> CW
    BW --> EX
    BW -.-> GWS
    BW -.-> GEI
    BW -.-> CSA

    CW --> SR
    CW --> RL
    ED -.-> EX

    SR -.-> GC
    RL --> WRJ

    WRJ --> W
    WRJ --> R
    WRJ --> J

    R --> CR
    R --> CTR
    R --> CE
    J -.-> AD

    classDef agent fill:#454545,stroke:#777777,color:#eeeeee,stroke-width:1.5px
    classDef tool fill:#454545,stroke:#777777,color:#eeeeee,stroke-width:1.5px,stroke-dasharray:5 4

    class BW,CW,EX,SR,RL,WRJ,W,R,J,CR,CTR,CE agent
    class GWS,GEI,CSA,GC,AD,ED tool

```

After several experiments using Ollama models (like, "gpt-oss:20b" or "nemotron-3.5-lightning:latest") 
or "big" OpenAI models ("gpt-5.6-terra" and "gpt-5.6-luna") certain manual tunings of the prompts in the nodes were made.
(And the results seem much better.)

