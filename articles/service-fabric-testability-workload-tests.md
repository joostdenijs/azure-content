<properties
   pageTitle="Custom Test Scenarios"
   description="How to harden your services against Graceful/Ungraceful failures"
   services="service-fabric"
   documentationCenter=".net"
   authors="anmolah"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="03/17/2015"
   ms.author="anmola"/>

# Simulate failures during service workloads

The scenarios shipping with Testability enable developers to not worry about dealing with individual faults. There are scenarios however, where an explicit interleaving of client workload and failures might be needed. The interleaving of client workload and faults, ensures that the service is actually performing some action when failure happens. Given the level of control testability provides these could be at precise points of the workload execution. This induction of faults at different states in the application can find bugs and improve quality.

## Sample custom scenario
This test shows a scenario where the interleaving of business workload with [graceful and ungraceful failures](service-fabric-testability-actions.md#graceful-vs-ungraceful-fault-actions). The faults should be induced while in the middle of service operations or compute for best results.

Lets walk through an example of a serice that exposes four workloads A, B, C and D. Each corresponds to a set of workflows and could be compute, storage or a mix. For the sake of simplicity we will abstract out the workloads in our example. The different faults executed in this example are.
  + RestartNode: Ungraceful fault to simulate a machine restart
  + RestartDeployedCodePackage: Ungraceful fault to simulate service host process crashes
  + RemoveReplica: Graceful fault to simulate replica removal
  + MovePrimary: Graceful fault to simulate replica moves triggered by Service Fabric Load Balancer

```csharp
// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll.

using System;
using System.Fabric;
using System.Fabric.Testability;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        // Replace these strings with the actual version for your cluster and appliction.
        string clusterConnection = "localhost:19000";
        Uri applicationName = new Uri("fabric:/samples/PersistentToDoListApp");
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");

        Console.WriteLine("Starting Workload Test...");
        try
        {
            RunTestAsync(clusterConnection, applicationName, serviceName).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Workload Test failed: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Workload Test completed successfully.");
        return 0;
    }

    public enum ServiceWorkloads
    {
        A,
        B,
        C,
        D
    }

    public enum ServiceFabricFaults
    {
        RestartNode,
        RestartCodePackage,
        RemoveReplica,
        MovePrimary,
    }

    public static async Task RunTestAsync(string clusterConnection, Uri applicationName, Uri serviceName)
    {
        // Create FabricClient with connection & security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);
        // Maximum time to wait for a service to stabilize
        TimeSpan maxServiceStabilizationTime = TimeSpan.FromSeconds(120);

        // How many loops of faults you want to execute
        uint testLoopCount = 20;
        Random random = new Random();

        for (var i = 0; i < testLoopCount; ++i)
        {
            var workload = SelectRandomValue<ServiceWorkloads>(random);
            // Start workload and while it is running go and induce some fault
            var workloadTask = RunWorkloadAsync(workload);

            // While task is executing induce faults into the service. It can be ungraceful faults like
            // RestartNode and RestartDeployedCodePackage or graceful faults like RemoveReplica or MovePrimary
            var fault = SelectRandomValue<ServiceFabricFaults>(random);

            // Create a replica selector which will select a Primary replica from the given service to test
            var replicaSelector = ReplicaSelector.PrimaryOf(PartitionSelector.RandomOf(serviceName));
            // Run the selected random fault
            await RunFaultAsync(applicationName, fault, replicaSelector, fabricClient);
            // Validate the health and stability of the service.
            await fabricClient.ServiceManager.ValidateServiceAsync(serviceName, maxServiceStabilizationTime);

            // Wait for the workload to complete successfully
            await workloadTask;
        }
    }

    private static async Task RunFaultAsync(Uri applicationName, ServiceFabricFaults fault, ReplicaSelector selector, FabricClient client)
    {
        switch (fault)
        {
            case ServiceFabricFaults.RestartNode:
                await client.ClusterManager.RestartNodeAsync(selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RestartCodePackage:
                await client.ApplicationManager.RestartDeployedCodePackageAsync(applicationName, selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RemoveReplica:
                await client.ServiceManager.RemoveReplicaAsync(selector, CompletionMode.Verify, false);
                break;
            case ServiceFabricFaults.MovePrimary:
                await client.ServiceManager.MovePrimaryAsync(selector.PartitionSelector);
                break;
        }
    }

    private static Task RunWorkloadAsync(ServiceWorkloads workload)
    {
        throw new NotImplementedException();
        // This is where you trigger and complete your service workload
        // Please note the faults induced while your service workload is running will
        // fault the Primary service hence you will need to reconnect to complete or check
        // the status of the workload
    }

    private static T SelectRandomValue<T>(Random random)
    {
        Array values = Enum.GetValues(typeof(T));
        T workload = (T)values.GetValue(random.Next(values.Length));
        return T;
    }
}
```
