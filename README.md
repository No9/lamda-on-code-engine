# lamda-on-code-engine
Run node.js lambda on IBM Cloud Code Engine Service.
This is a detailed set of steps to go from zero to a running lamda function on IBM Cloud.
 

## usage

1.  An IBM Cloud account 
    https://cloud.ibm.com/registration 

2.  Download the IBM Cloud cli and install the code engine plugin
    https://cloud.ibm.com/codeengine/cli 

3.  Login to IBM Cloud and target the US region and the default resource group.
    
    ```
    $ ibmcloud login
    $ ibmcloud target -r us-south
    $ ibmcloud target -g default
    $ ibmcloud cr region-set

    Choose a region
    1. ap-north ('jp.icr.io')
    2. ap-south ('au.icr.io')
    3. eu-central ('de.icr.io')
    4. global ('icr.io')
    5. uk-south ('uk.icr.io')
    6. us-south ('us.icr.io')
    
    6 
    ```

4.  Create a namespace in your container registry for Code Engine
    
    ```
    $ ibmcloud cr namespace-add ce-build
    ```
5.  Create a code engine project and select it.
    
    ```
    $ ibmcloud ce project create --name lambda-ce-project
    $ ibmcloud ce project select -n lambda-ce-project
    ```

6.  Create an API key to allow code engine to access the registry
    
    ```
    $ ibmcloud iam api-key-create cebuildkey -d "Code Engine Build Key" --file key_file
    ```

7.  Assign the key to the code engine instance
    
    ```
    $ ibmcloud ce registry create --name icr-ce-build --server us.icr.io --username iamapikey --password $(jq -r ."apikey" key_file)
    ```

8.  Create a build for the project
    
    ```
    $ ibmcloud ce build create --name lambda-ce-build --source https://github.com/No9/lamda-on-code-engine.git --strategy kaniko --image us.icr.io/ce-build/lambda-ce-app --registry-secret icr-ce-build --commit main
    ```
9.  Run the build

    ```
    $ ibmcloud ce buildrun submit --name lambda-ce-1 --build lambda-ce-build
    ```

10. Deploy the image onto Code Engine
    
    ```
    $ ibmcloud ce app create -n lambda-ce -i us.icr.io/ce-build/lambda-ce-app:latest -m 128Mi 
    ```

11. Once the service is deployed you should be able to post an an event to it that will be echoed back to you.

    ```
    $ curl -s -d '{"hello" : "world"}' https://END_POINT_DOMAIN/2015-03-31/functions/function/invocations
    {"hello" : "world"}
    ```
