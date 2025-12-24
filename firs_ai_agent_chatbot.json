{
  "nodes": [
    {
      "parameters": {
        "options": {
          "responseMode": "responseNodes"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.4,
      "position": [
        -1168,
        208
      ],
      "id": "6da7e1f7-50f4-47f6-885b-54668878829c",
      "name": "When chat message received",
      "webhookId": "d926501e-4322-44b1-97a4-0b460850e88c"
    },
    {
      "parameters": {
        "options": {
          "systemMessage": "You are an intelligent automation assistant for corporate finance department. \n\n\n\nYour job is to help users generate executive ready insights and next actions rules. \n\n\n\nYou must follow:\n1.- If the user is missing the report name or time frame ask verifying questions BEFORE answering. \n\n2.- Do not invent numbers if data is missing ask for it or say what do you need? \"\n\n3.- Always reply in the structure \n- answers (bulletized) \n- What I still need (if anything). \n- next question \n\nIf user says \"Hello\" or vague input, ask what report + timeframe they help with."
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3,
      "position": [
        -896,
        208
      ],
      "id": "7ae53f59-708f-454b-9f29-978426f215f3",
      "name": "AI Agent",
      "retryOnFail": true
    },
    {
      "parameters": {
        "projectId": {
          "__rl": true,
          "value": "vz-nonit-np-jaov-dev-fpasdo-0",
          "mode": "list",
          "cachedResultName": "vz-nonit-np-jaov-dev-fpasdo-0"
        },
        "options": {
          "maxOutputTokens": 800,
          "temperature": 0.2
        }
      },
      "type": "CUSTOM.lmChatGoogleVertexWIF",
      "typeVersion": 1,
      "position": [
        -944,
        432
      ],
      "id": "15cf68b9-1df5-4ed7-a32d-0a88590805c6",
      "name": "Google Vertex AI Chat Model with WIF",
      "credentials": {
        "googleWIFApi": {
          "id": "lRbLPJ3RiQvS6bmh",
          "name": "JAOV Service Account WIF(VertexAI)"
        }
      }
    },
    {
      "parameters": {
        "contextWindowLength": 10
      },
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [
        -784,
        432
      ],
      "id": "8b5615c9-220f-4178-b505-e0e3b83f96be",
      "name": "Simple Memory"
    },
    {
      "parameters": {
        "description": "Returns the list supported reports and what information is needed.",
        "jsCode": "return {\n  supported_reports: [\"CIPB\",\"OneEx Report\",\"CFO KPI\", \"Daily Device\"],\n  required_inputs: [\"input_name\",\"timeframe\"]\n  exaples: [\n  \"Summarize what changed in oneex report for 2025-12 vs 2025-11\",\n  \"Draft CIPB executive summary for Nov 2025 month-end\"\n  ]\n}\n"
      },
      "type": "@n8n/n8n-nodes-langchain.toolCode",
      "typeVersion": 1.3,
      "position": [
        -608,
        432
      ],
      "id": "c12ac88f-d7eb-4041-9e52-ba44ccead9df",
      "name": "get_supported_reports"
    },
    {
      "parameters": {
        "message": "={{ $json.output }}",
        "waitUserReply": false,
        "options": {
          "memoryConnection": false
        }
      },
      "type": "@n8n/n8n-nodes-langchain.chat",
      "typeVersion": 1,
      "position": [
        -480,
        208
      ],
      "id": "5d4e34a6-817c-4aaf-9f0f-f442c53b4be1",
      "name": "Respond to Chat"
    }
  ],
  "connections": {
    "When chat message received": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent": {
      "main": [
        [
          {
            "node": "Respond to Chat",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Google Vertex AI Chat Model with WIF": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Simple Memory": {
      "ai_memory": [
        [
          {
            "node": "AI Agent",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "get_supported_reports": {
      "ai_tool": [
        [
          {
            "node": "AI Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    }
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "47afbff3c3bd6cef1b07f625cd9e35241695ee198c9627afe8af9849f53cba16"
  }
}
