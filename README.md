# Azure IoT Hub Demo

Azure IoT features demo.

## Tools

You'll need to following software to run this demo:

* Azure CLI
* Azure CLI IoT Extension (`az extension add --name azure-cli-iot-ext`)

## Provisioning the Infrastructure

```powershell
# Login and setup your location variable
az login -u <username>
az configure --defaults location=eastus

# Group creation
$group="iotdemo"
az group create -n $group

# Create IoT Hub with free SKU (It cannot to be upgraded to Basic or Standard)
az iot hub create -n "iotdemohub" -g $group --sku F1

# Create the Device Provisioning Service (DPS)
az iot dps create -n iotdemodps -g $group

#Link the hub to the provisioning service
$hubConnectionString=az iot hub show-connection-string -n iotdemohub -o tsv
az iot dps linked-hub create -g $group --dps-name iotdemodps --connection-string $hubConnectionString

# Create a device:
az iot hub device-identity create --device-id test-device-01 --hub-name iotdemohub
```

## Sending Messages

```powershell
# D2C
az iot hub monitor-events -n iotdemohub999
az iot device send-d2c-message -n iotdemohub999 -d test-device-01 --data 'Hello from Azure CLI'

# C2D
az iot device c2d-message send -n iotdemohub999 -d test-device-01 --data 'Hello, device, from Azure CLI'
az iot device c2d-message receive -n iotdemohub999 -d test-device-01

# simulate device
az iot hub monitor-events -n iotdemohub999

az iot device simulate -n iotdemohub999 -d test-device-01 `
--data "Message from simulated device!" `
--msg-count 5
```

## Manual Enrollment

*These steps are shortened from [this Microsoft article](https://docs.microsoft.com/en-us/azure/iot-dps/quick-create-simulated-device-x509-csharp).*

1. `git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git`
2. `cd .\azure-iot-samples-csharp\provisioning\Samples\device\X509Sample`
3. Create the certificates: `powershell .\GenerateTestCertificate.ps1`
4. Configure individual enrollment:
```powershell
az iot dps enrollment create -g iotdemo --dps-name $iotdemodps999 `
--enrollment-id iothubx509device1 --attestation-type x509 --certificate-path certificate.cer
```
5. Get the DPS ID Scope: `az iot dps show -n iotdemodps999 --query "properties.idScope" -o tsv`
6. Send a message: `dotnet run <idScope>`

## Stream Analytics

1. Create a Stream Analytics Job using the Portal


## Security

Security concerns for IoT provisioning:

https://docs.microsoft.com/en-us/azure/iot-dps/concepts-security#hardware-security-module

## References

[Pluralsight Creating IoT Solutions](https://app.pluralsight.com/library/courses/microsoft-azure-iot-solutions-creating/table-of-contents)

[Managing DPS with CLI](https://docs.microsoft.com/en-us/azure/iot-dps/how-to-manage-dps-with-cli)

[Simulated device with CE](https://docs.microsoft.com/en-us/azure/iot-dps/quick-create-simulated-device-x509-csharp)

[X.509 Enrollment](https://docs.microsoft.com/en-us/azure/iot-dps/quick-enroll-device-x509-csharp)

[Send Telemetry](https://docs.microsoft.com/en-us/azure/iot-hub/quickstart-send-telemetry-dotnet)

[IoT client provisioning sample](https://github.com/MattHoneycutt/ps-create-iot-solutions/tree/master/device-provisioning-sample)

[IoT C# Samples](https://github.com/MattHoneycutt/ps-create-iot-solutions)
