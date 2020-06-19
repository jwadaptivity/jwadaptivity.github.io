---
layout: post
title: "Azure Function on Kubernetes"
date: 2020-06-19 13:00:00 -0000
categories: azure keda
---
<h2>Azure Functions on Kubernetes</h2>

This blog post is an explation with a working example of how to containerise your azure function, deploy it to a kubernetes cluster, and scale it automatically using <a href="https://keda.sh/">KEDA</a>. 
<blockquote style="background: #f5f5fb;padding:12px;"> "KEDA is a Kubernetes-based event-driven autoscaler. KEDA determines how any container in Kubernetes should be scaled based on the number of events that need to be processed." </blockquote>
The azure function in this demo uses a Queue Trigger which will scale within the kuberenetes cluster based on the number of messages in the queue, reacting to events rather than the traditonal approach of reacting to requests. Scaling can go from 0 to N instances by leveraging the Kuberenetes autoscaler.

You're expected to have familiarity with Docker, Azure, .NET and Kuberentes for the tutorial.

<h3>Azure Function</h3>
An azure function is a serverless application usually deployed within an Azure subscription. To containerise the application we are using the Azure Function runtime image to enable us to execute Azure functions within a docker container. The more common solution would be to host the Azure Function on the Azure platform where all infrasturcute and scaling is taken care of.
 
<h3>Kubernetes</h3>
Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.
Most popular cloud providers now offer a kubernetes experience, for Azure it is AKS, this demo will use the desktop version known as Minikube, explained later. Keda is available on all popular cloud providers 

<h3>Demo</h3> 
<h4>Pre-requists</h4>
To complete the demo you will need the following tools
<div class="highlight"><pre class="highlight">
<span class="p">-</span> <a href="https://kubernetes.io/docs/tasks/tools/install-minikube/">MiniKube</a>
<span class="p">-</span> <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cbash">Azure Functions Core Tools</a>
<span class="p">-</span> <a href="https://docs.docker.com/docker-for-windows/install/">Docker Desktop</a>
<span class="p">-</span> <a href="https://azure.microsoft.com/en-gb/free/">Azure Subscription</a>
<span class="p">-</span> <a href="https://code.visualstudio.com/">VS Code</a>
</pre></div>

Firstly we need to create an azure function. To do this we can use VS code to spin up the boilder plate for us. Just make sure you have the 
<a href="https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions">Azure functions extension</a>. Once that is installed just click the picture as below and follow the instructions. Just make sure to select .NET core application and QueueTrigger when prompted.
You will be prompted for the azure storage account that the QueueFunction will trigger from so create that in the portal and make a note of the exception string

![Keda in action](/images/createNewProject.png)

Once you have finished setting up your project you should have a function ready to go as below:
![Keda in action](/images/func.cs.png)

The sleep timer has been added to help illustrate the scaling capabilites of keda later on in the demo. 
Now that the function app is created, from the same directory use the following command, which is from the Azure function tools.

```powershell
func init docker only 
```

This will generate a dockerfile which we will now use to build our function app image using the function runtime.
```powershell
docker build . -t "queue-function"  
```
Now you need to tage the image and upload it to your registry, in this example I used Azure Container Registry which requires a few more steps later on regarding service principals. For ease I would recommend docker hub. You need to login to your docker registry before executing the following commands.

```powershell
docker tag queue-function <registry>/queue-function:latest 
docker push <registry>/queue-function:latest
```

So at this point we have our azure function boxed up in a container image and saved to a repoistory. The next step is to get KEDA running on Kubernetes and use our docker image as the function. So firstly, lets install KEDA onto kuberenetes using this command:

```powershell
func kubernetes install --namespace keda
```
Once this is complete we can then see it up and running
```powershell
kubectl --namespace keda get svc
```
![Keda in action](/images/k8_get_Svc.png)

Finally we deploy our image onto kubernetes cluster, and the func cli commands KEDA to set up autoscaling rules and pods automatically 
```powershell
func kubernetes deploy --name queue-function --registry jackdevkeda.azurecr.io  
```
And if you add some messages to the queue as below
![Keda in action](/images/queue.png)

I created a small console application to push 1000s of messages to the queue at once, you can see KEDA in action below, the number of pods being created to handle the messages automatically:

![Keda in action](/images/keda_in_Action.gif)

<h3>Azure Container Registry</h3>
I deployed my docker image to the Azure Container Registry (ACR) rather than the docker hub, out of curiosity. This came with permission issues which I overcame by creating a service prinicpal and configuring the kubernetes service account to use the credentials.

So firstly you need to create a service principal, I used the following script. Just make sure to capture the AppID and PWD variables.
```powershell
$ACR_NAME="<repo>.azurecr.io"
$SERVICE_PRINCIPAL_NAME="acr-service-principal"

$ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
$SP_PASSWD=$(az ad sp create-for-rbac --name http://$SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role acrpull --query password --output tsv)
$SP_APP_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

```
Then I created the kubernetes secret to store the credential as below 
```powershell 
kubectl create secret docker-registry acrcred --docker-server= <repo>.azurecr.io  --docker-username=AppID
   --docker-password=PWD
#Output the kuberenetes service account to a file for editing
 kubectl get serviceaccount default -o yaml > ./sa.yaml
```
add this to the output yaml file

```yaml
imagePullSecrets:
- name: regcred
```
```powershell
#Apply changes to kubernetes 
kubectl replace serviceaccount default -f ./sa.yaml
```
This will allow your cluster to pull images from the ACR registry.

<h3>Tips</h3>
If you edit the host file (below) of your function app you can make sure only one mesage is eaten at a time, so the app consumes them slowly and we can watch the caling in action. Remember to rebuild and deploy your image after changing the host file. And then restart the kubernetes pods.

```json
{
    "version": "2.0",
    "extensions": {
        "queues": {
            "batchSize": 1
        }
    },
    "logging": {
        "applicationInsights": {
            "samplingExcludedTypes": "Request",
            "samplingSettings": {
                "isEnabled": true
            }
        }
    }
}
``` 
Below is a console application you can quickly run to send 1000s of messages onto your queue for testing.
```c#
class Program
    {
        static async Task Main(string[] args)
        {
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse("string");
            CloudQueueClient queuClient = storageAccount.CreateCloudQueueClient();
            CloudQueue queue = queuClient.GetQueueReference("kedaqueue");
            for(int i = 0; i < 1000 ; i++)
            {
                await queue.AddMessageAsync(new CloudQueueMessage($"Hello for the {i}  time"));
            } 
        }
    }
```