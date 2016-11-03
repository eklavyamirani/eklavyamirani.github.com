For the last few months, I have been using ADF for ETLing data from a DB2 database over to a summarized Azure Sql Database. It has been quite a ride, really. One of the interesting bits of code I had to write was to automate the deployment of the configuration files for ADF over to 5 different environments.

An easy way for debugging is to either use the portal directly, or use the publish tool via Visual Studio. But again, it is *debug* only.

We need powershell commandlets, or someone to manually upload the files to the ADF instance, and naturally, the fork in this road was solved by laziness to upload 100+ files, one by one *cringes*.

According to Microsoft's offering, the following powershell commandlets can be used to upload these files via powershell:
  1. New-AzureRmDataFactoryLinkedService
  2. New-AzureRmDataFactoryDataSet
  3. New-AzureRmDataFactoryPipeline

This means that we need a way to segregate our json configs into their respective categories:
  1. Linked Services
  2. Datasets
  3. Pipelines

~Also, note that the order of deploying/cleaning up these files is also important, and if done out of order, it will not work.

Linked Services -> Datasets -> Pipelines~

This means, if we are able to somehow point which config is of what type, we can create a script that will upload all these files in correct order to the ADF instance.

One way to do this is to create seperate directories for Linked Services, Datasets and Pipelines, and feed everything present in these directories to the commandlet. ~Something like~ Exactly like:

~~~~
$linkedServices = @(Get-ChildItem ".\Linked Services" -Filter *.json)

    for ($i = 0; $i -lt $linkedServices.Length; ++$i) {
        New-AzureRmDataFactoryLinkedService -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -File $linkedServices[$i].PSPath -Force
    }
~~~~

The .dfproj for ADF seperates the Linked Services, Datasets and Pipelines into seperate virtual directories, but physically everything is in one single directory. 

But we want these to be seperated into physical directories, and it doesn't look like it is too difficult, just write a post build XCopy and we're done. TADA! **Nope**

The dfproj does not have an option to write post build scripts, or pretty much any hooks into the build process. Not without playing with MSBuild anyway.

Our deployment environments have different sources and schemas. This means that while the datasets can be reused we would have seperate pipelines and linked services for each of our environments. [1]

Here is what the linked services look like:
  1. DB2LinkedService.json
  + DB2LinkedService.test.json
  + DB2LinkedService.int.json
  + DB2LinkedService.preprod.json
  + DB2LinkedService.prod.json

The actual name of all these files however is DB2LinkedService.json. This allows the 100+ datasets to stay common, instead of aggregating to 500+ datasets. But the challenge is, this fails ADF validations because of conflicting names. All files hould have a unique name. Obviously.

Because these are just copies of each other, with only minor variations, it would suffice to only validate one of them, and not the others. And in case those minor variations, i.e. the connection details are invalid, they would anyway fail during deployment.

Excluding the duplicate files from the project meant that to modify any of them, we'd have to step out of Visual Studio. *ouch*

Instead, I included them as a non-building project item. Here is how:
~~~~
<ItemGroup>
    <None Include="DB2LinkedService.test.json" />
    <None Include="DB2LinkedService.int.json" />
    <None Include="DB2LinkedService.preprod.json" />
    <None Include="DB2LinkedService.prod.json" />
  </ItemGroup>
~~~~

And Visual Studio is pretty smart, it will still place these files under linked services!
[Image here]

Now our project structure looks good, and we come back to the original problem. How do we add post build deployment scripts?

MSBuild is actually quite powerful. It provides various tasks <https://msdn.microsoft.com/en-us/library/7z253716.aspx> and out of these, we are interested in copying files to a directory (Copy Task), and tearing down this directory (RemoveDir Task).

Now all we need is a hook into the Build and Clean targets of the dfproj.

Because the DataFactory.targets file (MSBuild file that contains the tasks to build a data factory project, also imports Microsoft.Common.targets, it defines the 'AfterTargets="Build"' attribute on any custom target that you create. This means that this target will be executed after the Build target.


~~~~WIP~~~~


*[1] Linked services provide support for different schema names, and ideally, pipelines should not contain schema names. In our case we use a mixture of schemas in our read queries, it is not feasible to link the activities to multiple linked services.*
