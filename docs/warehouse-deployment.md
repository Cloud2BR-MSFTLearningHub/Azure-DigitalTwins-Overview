# Warehouse deployment

This walkthrough deploys the repository's warehouse graph through the Azure portal and Azure CLI. It creates the Azure Digital Twins instance, registers models, instantiates twins, connects relationships, and prepares a telemetry integration.

## Prerequisites

- Azure subscription with permission to create and manage the required resources.
- Azure CLI and the `azure-iot` extension.
- An Azure Digital Twins instance in a supported region.
- DTDL authoring tools such as Visual Studio Code with a DTDL extension.

!!! important
    Grant a deployment identity the Azure Digital Twins Data Owner role before it creates models, twins, or relationships. Role assignments can take time to propagate.

## 1. Create the Azure Digital Twins instance

In the Azure portal, search for **Azure Digital Twins**, select **Create**, choose a resource group and region, and record the deployed host URL.

<img width="900" alt="Create an Azure Digital Twins resource in the Azure portal" src="https://github.com/user-attachments/assets/21ae1100-12ec-408c-8e7a-e163bbfed95a" />

## 2. Register DTDL models

Start with the model files in [models/](https://github.com/Cloud2BR-MSFTLearningHub/Azure-DigitalTwins-Overview/tree/main/models). Azure CLI can register a model directly from a local JSON file:

```bash
az dt model create \
  --dt-name <instance-name> \
  --models models/warehouse.json
```

## 3. Create twins and relationships

Create twin instances from the registered models, then connect the graph. Use meaningful IDs so that operational tooling can identify entities unambiguously.

```bash
az dt twin create \
  --dt-name <instance-name> \
  --twin-id shelf01 \
  --model-id "dtmi:com:example:Shelf;1"

az dt twin relationship create \
  --dt-name <instance-name> \
  --twin-id zone01 \
  --relationship-id zone01_shelf01 \
  --target shelf01 \
  --relationship-name contains
```

## 4. Connect device telemetry

Use Azure IoT Hub for device connectivity and Azure Functions or Event Grid to transform and route messages into twin updates. A shelf signal could look like this:

```json
{
  "currentLoad": 42
}
```

The processor should map the incoming device identity to the correct twin and patch the intended property or telemetry field. Add validation, retry handling, and observability before using the pattern in production.

## 5. Inspect the graph

Use Azure Digital Twins Explorer to explore relationships and run queries.

```sql
SELECT *
FROM digitaltwins
WHERE IS_OF_MODEL('dtmi:com:example:Shelf;1')
```

<img width="900" alt="Azure Digital Twins Explorer graph interface" src="https://github.com/user-attachments/assets/8a19883b-e76e-4868-920b-a27471a520df" />

Continue with [Automation and operations](automation-and-operations.md) to use the repository scripts, Terraform reference infrastructure, and diagnostics.