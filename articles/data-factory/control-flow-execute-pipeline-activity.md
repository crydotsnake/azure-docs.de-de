---
title: Aktivität „Pipeline ausführen“ in Azure Data Factory
description: Erfahren Sie, wie Sie die Aktivität „Pipeline ausführen“ verwenden können, um eine Data Factory-Pipeline über eine andere Data Factory-Pipeline aufzurufen.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 4bd667a2302136b5e12d2e4e548c9e8863715621
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81415281"
---
# <a name="execute-pipeline-activity-in-azure-data-factory"></a>Aktivität „Pipeline ausführen“ in Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Mit der Aktivität „Pipeline ausführen“ kann eine Data Factory-Pipeline eine andere Pipeline aufrufen.



## <a name="syntax"></a>Syntax

```json
{
    "name": "MyPipeline",
    "properties": {
        "activities": [
            {
                "name": "ExecutePipelineActivity",
                "type": "ExecutePipeline",
                "typeProperties": {
                    "parameters": {                        
                        "mySourceDatasetFolderPath": {
                            "value": "@pipeline().parameters.mySourceDatasetFolderPath",
                            "type": "Expression"
                        }
                    },
                    "pipeline": {
                        "referenceName": "<InvokedPipelineName>",
                        "type": "PipelineReference"
                    },
                    "waitOnCompletion": true
                 }
            }
        ],
        "parameters": [
            {
                "mySourceDatasetFolderPath": {
                    "type": "String"
                }
            }
        ]
    }
}
```

## <a name="type-properties"></a>Typeigenschaften

Eigenschaft | BESCHREIBUNG | Zulässige Werte | Erforderlich
-------- | ----------- | -------------- | --------
name | Name der Aktivität „Pipeline ausführen“. | String | Ja
type | Muss auf **ExecutePipeline** festgelegt werden. | String | Ja
pipeline | Pipelineverweis auf die abhängige Pipeline, die diese Pipeline aufruft. Ein Pipelineverweisobjekt verfügt über zwei Eigenschaften: **referenceName** und **type**. Die Eigenschaft „referenceName“ gibt den Namen des Pipelineverweises an. Die Eigenschaft „type“ muss auf „PipelineReference“ festgelegt werden. | PipelineReference | Ja
parameters | Parameter, die an die aufgerufene Pipeline übergeben werden sollen | Ein JSON-Objekt, das Parameternamen Argumentwerten zuordnet | Nein
waitOnCompletion | Definiert, ob die Aktivitätsausführung wartet, bis die Ausführung der abhängigen Pipeline abgeschlossen ist. Der Standardwert ist "false". | Boolean | Nein

## <a name="sample"></a>Beispiel
In diesem Szenario gibt es zwei Pipelines:

- **Masterpipeline**: Diese Pipeline weist eine Aktivität „Pipeline ausführen“ auf, die die aufgerufene Pipeline aufruft. Die Masterpipeline nutzt zwei Parameter: `masterSourceBlobContainer`, `masterSinkBlobContainer`.
- **Aufgerufene Pipeline**: Diese Pipeline weist eine Kopieraktivität auf, die Daten aus einer Azure-Blobquelle in eine Azure-Blobsenke kopiert. Die aufgerufene Pipeline nutzt zwei Parameter: `sourceBlobContainer`, `sinkBlobContainer`.

### <a name="master-pipeline-definition"></a>Definition der Masterpipeline

```json
{
  "name": "masterPipeline",
  "properties": {
    "activities": [
      {
        "type": "ExecutePipeline",
        "typeProperties": {
          "pipeline": {
            "referenceName": "invokedPipeline",
            "type": "PipelineReference"
          },
          "parameters": {
            "sourceBlobContainer": {
              "value": "@pipeline().parameters.masterSourceBlobContainer",
              "type": "Expression"
            },
            "sinkBlobContainer": {
              "value": "@pipeline().parameters.masterSinkBlobContainer",
              "type": "Expression"
            }
          },
          "waitOnCompletion": true
        },
        "name": "MyExecutePipelineActivity"
      }
    ],
    "parameters": {
      "masterSourceBlobContainer": {
        "type": "String"
      },
      "masterSinkBlobContainer": {
        "type": "String"
      }
    }
  }
}

```

### <a name="invoked-pipeline-definition"></a>Definition der aufgerufenen Pipeline

```json
{
  "name": "invokedPipeline",
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "name": "CopyBlobtoBlob",
        "inputs": [
          {
            "referenceName": "SourceBlobDataset",
            "type": "DatasetReference"
          }
        ],
        "outputs": [
          {
            "referenceName": "sinkBlobDataset",
            "type": "DatasetReference"
          }
        ]
      }
    ],
    "parameters": {
      "sourceBlobContainer": {
        "type": "String"
      },
      "sinkBlobContainer": {
        "type": "String"
      }
    }
  }
}

```

**Verknüpfter Dienst**

```json
{
    "name": "BlobStorageLinkedService",
    "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=*****;AccountKey=*****"
    }
  }
}
```

**Quelldataset**
```json
{
    "name": "SourceBlobDataset",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@pipeline().parameters.sourceBlobContainer",
        "type": "Expression"
      },
      "fileName": "salesforce.txt"
    },
    "linkedServiceName": {
      "referenceName": "BlobStorageLinkedService",
      "type": "LinkedServiceReference"
    }
  }
}
```

**Senkendataset**
```json
{
    "name": "sinkBlobDataset",
    "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": {
        "value": "@pipeline().parameters.sinkBlobContainer",
        "type": "Expression"
      }
    },
    "linkedServiceName": {
      "referenceName": "BlobStorageLinkedService",
      "type": "LinkedServiceReference"
    }
  }
}
```

### <a name="running-the-pipeline"></a>Ausführen der Pipeline

Um die Masterpipeline in diesem Beispiel auszuführen, werden die folgenden Werte für die Parameter „masterSourceBlobContainer“ und „masterSinkBlobContainer“ übergeben: 

```json
{
  "masterSourceBlobContainer": "executetest",
  "masterSinkBlobContainer": "executesink"
}
```

Die Masterpipeline leitet diese Werte an die aufgerufene Pipeline weiter, wie im folgenden Beispiel gezeigt: 

```json
{
    "type": "ExecutePipeline",
    "typeProperties": {
      "pipeline": {
        "referenceName": "invokedPipeline",
        "type": "PipelineReference"
      },
      "parameters": {
        "sourceBlobContainer": {
          "value": "@pipeline().parameters.masterSourceBlobContainer",
          "type": "Expression"
        },
        "sinkBlobContainer": {
          "value": "@pipeline().parameters.masterSinkBlobContainer",
          "type": "Expression"
        }
      },

      ....
}

```
## <a name="next-steps"></a>Nächste Schritte
Weitere Informationen finden Sie unter anderen Ablaufsteuerungsaktivitäten, die von Data Factory unterstützt werden: 

- [ForEach-Aktivität](control-flow-for-each-activity.md)
- [Aktivität „Metadaten abrufen“](control-flow-get-metadata-activity.md)
- [Lookup-Aktivität](control-flow-lookup-activity.md)
- [Webaktivität](control-flow-web-activity.md)
