---
title: Implemente uma solução MongoDB altamente disponível para O Azure e Azure Stack Hub
description: Saiba como implementar uma solução MongoDB altamente disponível para o Azure e o Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 624f032def509d8e42d55807d72176e5fce85910
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901512"
---
# <a name="deploy-a-highly-available-mongodb-solution-across-two-azure-stack-hub-environments"></a><span data-ttu-id="b8d2a-103">Implementar uma solução MongoDB altamente disponível em dois ambientes Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="b8d2a-103">Deploy a highly available MongoDB solution across two Azure Stack Hub environments</span></span>

<span data-ttu-id="b8d2a-104">Este artigo irá passar por uma implementação automatizada de um cluster básico altamente disponível (HA) MongoDB com um site de recuperação de desastres (DR) em dois ambientes do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="b8d2a-105">Para saber mais sobre o MongoDB e a alta disponibilidade, consulte [os Membros conjuntos de réplicas.](https://docs.mongodb.com/manual/core/replica-set-members/)</span><span class="sxs-lookup"><span data-stu-id="b8d2a-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="b8d2a-106">Nesta solução, criará um ambiente de amostra para:</span><span class="sxs-lookup"><span data-stu-id="b8d2a-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="b8d2a-107">Orquestrou uma implantação através de dois Hubs Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="b8d2a-108">Use o Docker para minimizar problemas de dependência com perfis AZure API.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="b8d2a-109">Implemente um cluster mongoDB básico altamente disponível com um local de recuperação de desastres.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="b8d2a-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="b8d2a-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="b8d2a-111">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="b8d2a-112">O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que permite construir e implementar aplicações híbridas em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="b8d2a-113">O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="b8d2a-114">As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="b8d2a-115">Arquitetura para MongoDB com Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="b8d2a-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![arquitetura mongoDB altamente disponível em Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="b8d2a-117">Pré-requisitos para MongoDB com Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="b8d2a-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="b8d2a-118">Dois sistemas integrados Azure Stack Hub (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="b8d2a-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="b8d2a-119">Esta implementação não funciona no Kit de Desenvolvimento de Pilhas Azure (ASDK).</span><span class="sxs-lookup"><span data-stu-id="b8d2a-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="b8d2a-120">Para saber mais sobre o Azure Stack Hub, veja [o que é O Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="b8d2a-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="b8d2a-121">Uma subscrição de inquilino em cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="b8d2a-122">**Tome nota de cada ID de subscrição e do ponto final do Azure Resource Manager para cada Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="b8d2a-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="b8d2a-123">Um diretor de serviço Azure Ative (Azure AD) que tem permissões para a subscrição do inquilino em cada Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="b8d2a-124">Você pode precisar de criar dois principais de serviço se os Azure Stack Hubs forem implantados contra diferentes inquilinos AD Azure.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="b8d2a-125">Para aprender a criar um principal de serviço para o Azure Stack Hub, consulte [utilizar uma identidade de aplicação para aceder aos recursos do Azure Stack Hub.](/azure-stack/user/azure-stack-create-service-principals)</span><span class="sxs-lookup"><span data-stu-id="b8d2a-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="b8d2a-126">**Tome nota da identificação de cada pedido do diretor de serviço, segredo do cliente e nome do inquilino (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="b8d2a-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="b8d2a-127">Ubuntu 16.04 sindicado para cada Azure Stack Hub's Marketplace.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="b8d2a-128">Para saber mais sobre a sindicalização do mercado, consulte [itens de Mercado de Descarregamento para Azure Stack Hub.](/azure-stack/operator/azure-stack-download-azure-marketplace-item)</span><span class="sxs-lookup"><span data-stu-id="b8d2a-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="b8d2a-129">[Docker para Windows](https://docs.docker.com/docker-for-windows/) instalado na sua máquina local.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="b8d2a-130">Obtenha a imagem do Docker</span><span class="sxs-lookup"><span data-stu-id="b8d2a-130">Get the Docker image</span></span>

<span data-ttu-id="b8d2a-131">As imagens do Docker para cada implementação eliminam problemas de dependência entre diferentes versões do Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="b8d2a-132">Certifique-se de que o Docker para o Windows está a utilizar recipientes Windows.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="b8d2a-133">Executar o seguinte comando num pedido de comando elevado para obter o recipiente Docker com os scripts de implantação.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="b8d2a-134">Implementar os clusters</span><span class="sxs-lookup"><span data-stu-id="b8d2a-134">Deploy the clusters</span></span>

1. <span data-ttu-id="b8d2a-135">Uma vez puxada com sucesso a imagem do recipiente, inicie a imagem.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="b8d2a-136">Uma vez iniciado o contentor, receberá um terminal PowerShell elevado no recipiente.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="b8d2a-137">Mude os diretórios para chegar ao script de implementação.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="b8d2a-138">Executar a implantação.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-138">Run the deployment.</span></span> <span data-ttu-id="b8d2a-139">Forneça credenciais e nomes de recursos sempre que necessário.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="b8d2a-140">HA refere-se ao Azure Stack Hub onde o cluster HA será implantado.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="b8d2a-141">Dr refere-se ao Azure Stack Hub onde o cluster DR será implantado.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. <span data-ttu-id="b8d2a-142">Escreva `Y` para permitir a instalação do fornecedor NuGet, que irá dar início aos módulos "2018-03-01-híbridos" a instalar.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="b8d2a-143">Os recursos ha serão implantados primeiro.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-143">The HA resources will deploy first.</span></span> <span data-ttu-id="b8d2a-144">Monitorize a colocação e espere que termine.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="b8d2a-145">Assim que tiver a mensagem indicando que a implementação de HA está terminada, pode verificar o portal do HA Azure Stack Hub para ver os recursos utilizados.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="b8d2a-146">Continue com a implementação de recursos DR e decida se gostaria de ativar uma caixa de salto no DR Azure Stack Hub para interagir com o cluster.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="b8d2a-147">Aguarde que a implementação de recursos DR termine.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="b8d2a-148">Uma vez terminada a implantação de recursos DR, saia do recipiente.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="b8d2a-149">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="b8d2a-149">Next steps</span></span>

- <span data-ttu-id="b8d2a-150">Se ativou a caixa de salto VM no DR Azure Stack Hub, pode ligar-se via SSH e interagir com o cluster MongoDB instalando o mongo CLI.</span><span class="sxs-lookup"><span data-stu-id="b8d2a-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="b8d2a-151">Para saber mais sobre a interação com o MongoDB, consulte [a Mongo Shell.](https://docs.mongodb.com/manual/mongo/)</span><span class="sxs-lookup"><span data-stu-id="b8d2a-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="b8d2a-152">Para saber mais sobre aplicações híbridas em nuvem, consulte [a Hybrid Cloud Solutions.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="b8d2a-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="b8d2a-153">Modifique o código para esta amostra no [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="b8d2a-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
