---
title: "Flexing the Bicep"
date: "2023-12-27"
spoiler: "More powerful than ARM templates, but it's not Terraform."
---

After changing jobs this year, I have moved from working with AWS to working with Azure. It's not the first time I've used Azure. My research group in grad school used Azure. Bill Gates himself visited my research advisor and showered us with free credits. It was like being a kid in a candy store.

Returning to the candy store, it felt a little like using the WayBackMachine. The core front end code base appears to still be written in jQuery. Now, I can understand Oracle's business mentality of keeping their UI/UX in the 90's. Nothing says "we are too big to care" better than a GUI from 1980. Azure, though, is supposed to be competing with AWS for cloud domination. It's not a commodity yet, at least for the big players.

It's insane that they are advocating for LLMs to take over the work of programmers, but nobody in the organization has figured out how to refactor off of old javascript frameworks in 15 years. AWS / New Relic / Confluent / Google... all these cloud services strive to show real time data and allow a user to quickly get to what is going wrong.

The smell of technical debt is strong on the Azure Side. The Portal is sluggish, you need to press the refresh button to see most updates. The Azure CLI sent me back a web page as an error, because maybe something is written in .NET Framework still? Issues and errors I've seen make me question the back end, and how well Azure is managed as a whole. Articles like these, for example: [Has Microsoft cut security corners once too often?](https://www.computerworld.com/article/3704132/has-microsoft-cut-security-corners-once-too-often.html) and [Microsoft Signing Key Stolen by Chinese](https://www.schneier.com/blog/archives/2023/08/microsoft-signing-key-stolen-by-chinese.html) cut to the core of my concerns.

But, we'll brush those aside, as there are some nice features inside Azure. I focus on Terraform, Powershell, and Bash to design Infrastructure as code, so I can ignore the Azure Portal as much as possible in my everyday work.

ARM templates are a great way to get started with Azure, but they are not the best way to manage your infrastructure. Having to conform to the JSON specification makes them unwieldy for variables and complex conditional statements.

Bicep is the promising outgrowth of 'ARM Templates', allowing you to write infrastructure as code with a single purpose domain-specific language (DSL). Microsoft's DNA has always been focused on building languages and DSLs, and it shows. Anders Hejlsberg is a bit of a legend when it comes to language architecture. He's got a knack for creating languages like Typescript that are extensible and flexible, yet precise. Not sure he has much to do with Bicep's development, but [I know he gave them good feedback](https://github.com/Azure/bicep/issues/10121).

For example. I can build a virtual network with the following code:

```js
param location string = resourceGroup().location

resource virtualNetwork 'Microsoft.Network/virtualNetworks@2023-06-01' = {
  name: 'vnet1'
  location: location

  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }

    subnets: [
      {
        name: 'appSubnet'
        properties: {
          addressPrefix: '10.0.1.0/24'
        }
      }
      {
        name: 'storageSubnet'
        properties: {
          addressPrefix: '10.0.2.0/24'
        }
      }
    ]
  }
}

```

From there, I can deploy this virtual network to any Azure region by passing the location of the resource group I want to deploy to:

`az deployment group create --resource-group test-rg --template-file virtual-network.bicep`. With the use of modules you can have a nice collection of resources ready to deploy.

A big difference between Terraform and Bicep, is that Bicep is stateless. There is no way to 'roll-back' or delete resources that you've just deployed with a Bicep file.

I asked GPT-4 how to delete the virtual network created above with Bicep and it responded:

```markdown
To delete the vitual network resource in Bicep,
you can use the `delete`` keyword:

resource deleteNetwork 'Microsoft.Network/virtualNetworks@2021-08-01' = {
name: vnet.name
location: resourceGroup().location
delete: true
}

The key points:

- Create a new resource with the same type
  (Microsoft.Network/virtualNetworks)
- Specify the name property to target the virtual network you want to delete
- Set the location to match the existing network
- Use `delete: true`` to indicate this resource should delete the existing
  virtual network
```

Which is a great idea, maybe I could PR this request to the Azure Bicep team and see what they think. Unfortunately, it's also **wrong**. There are no `delete` fields in Bicep resources. The best way I could find to do this was using the [Complete Mode](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-modes#complete-mode) in the Microsoft Documentation. In complete mode, Resource Manager deletes resources that exist in the resource group but aren't specified in the template.

This is a great feature, but it will require me to shift from my previous stateful toolset with Terraform. I loved the ability with Terraform to stand up and tear down large "infrastructure ideas" quickly and easily. I'm going to loose that ability, but I'll be honest. Production resources need stability, and I often found Terraform's stateful methodology to be tricky to maintain. [Ricard Bejarano](https://www.bejarano.io/terraform-stateless/) explains why Terraform should have remained stateless better than I can, so I'll leave it at that.
