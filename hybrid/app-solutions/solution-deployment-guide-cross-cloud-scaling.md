---
title: Implemente uma app que escala a nuvem cruzada em Azure e Azure Stack Hub
description: Saiba como implementar uma aplicação que escala a nuvem cruzada no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886820"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="dd2ea-103">Implemente uma aplicação que escala a nuvem cruzada usando O Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="dd2ea-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="dd2ea-104">Aprenda a criar uma solução cross-cloud para fornecer um processo manualmente desencadeado para mudar de uma aplicação web hospedada Azure Stack para uma aplicação web hospedada a Azure com autoscaling através do gestor de tráfego.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="dd2ea-105">Este processo garante uma utilidade de nuvem flexível e escalável quando está carregada.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="dd2ea-106">Com este padrão, o seu inquilino pode não estar pronto para executar a sua app na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="dd2ea-107">No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária no seu ambiente no local para lidar com picos na procura da app.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="dd2ea-108">O seu inquilino pode fazer uso da elasticidade da nuvem pública com a sua solução no local.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="dd2ea-109">Nesta solução, você construirá um ambiente de amostra para:</span><span class="sxs-lookup"><span data-stu-id="dd2ea-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="dd2ea-110">Crie uma aplicação web multi-nó.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="dd2ea-111">Configure e gere o processo de Implementação Contínua (CD).</span><span class="sxs-lookup"><span data-stu-id="dd2ea-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="dd2ea-112">Publique a aplicação web no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="dd2ea-113">Criar uma libertação.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-113">Create a release.</span></span>
> - <span data-ttu-id="dd2ea-114">Aprenda a monitorizar e a rastrear as suas implementações.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="dd2ea-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="dd2ea-116">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="dd2ea-117">O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que permite construir e implementar aplicações híbridas em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="dd2ea-118">O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="dd2ea-119">As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="dd2ea-120">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="dd2ea-120">Prerequisites</span></span>

- <span data-ttu-id="dd2ea-121">Subscrição do Azure.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-121">Azure subscription.</span></span> <span data-ttu-id="dd2ea-122">Se necessário, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="dd2ea-123">Um sistema integrado Azure Stack Hub ou implementação do Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="dd2ea-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="dd2ea-124">Para obter instruções sobre a instalação do Azure Stack Hub, consulte [instalar o ASDK](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="dd2ea-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="dd2ea-125">Para um script de automatização pós-implantação ASDK, vá a: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="dd2ea-126">Esta instalação pode requerer algumas horas para ser concluída.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="dd2ea-127">Implementar serviços paaS do Serviço de [Aplicações](/azure-stack/operator/azure-stack-app-service-deploy.md) para o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="dd2ea-128">[Crie planos/ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) dentro do ambiente Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="dd2ea-129">[Crie subscrição de inquilino](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) dentro do ambiente Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="dd2ea-130">Crie uma aplicação web dentro da subscrição do inquilino.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="dd2ea-131">Tome nota do novo URL da aplicação web para utilização posterior.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="dd2ea-132">Implementar a máquina virtual Azure Pipelines (VM) dentro da subscrição do inquilino.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="dd2ea-133">É necessário windows server 2016 VM com .NET 3.5.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="dd2ea-134">Este VM será construído na subscrição do inquilino no Azure Stack Hub como o agente de construção privada.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="dd2ea-135">[O Windows Server 2016 com imagem VM SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) está disponível no Azure Stack Hub Marketplace.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="dd2ea-136">Se esta imagem não estiver disponível, trabalhe com um Azure Stack Hub Operator para garantir que é adicionada ao ambiente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="dd2ea-137">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="dd2ea-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="dd2ea-138">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="dd2ea-138">Scalability</span></span>

<span data-ttu-id="dd2ea-139">O componente-chave do escalonamento de nuvens cruzadas é a capacidade de fornecer escalas imediatas e a pedido entre infraestruturas de nuvem públicas e no local, fornecendo um serviço consistente e fiável.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="dd2ea-140">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="dd2ea-140">Availability</span></span>

<span data-ttu-id="dd2ea-141">Certifique-se de que as aplicações implementadas localmente são configuradas para alta disponibilidade através da configuração de hardware no local e implementação de software.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="dd2ea-142">Capacidade de gestão</span><span class="sxs-lookup"><span data-stu-id="dd2ea-142">Manageability</span></span>

<span data-ttu-id="dd2ea-143">A solução cross-cloud garante uma gestão perfeita e uma interface familiar entre ambientes.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="dd2ea-144">O PowerShell é recomendado para a gestão de plataformas cruzadas.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="dd2ea-145">Dimensionamento entre clouds</span><span class="sxs-lookup"><span data-stu-id="dd2ea-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="dd2ea-146">Obtenha um domínio personalizado e configuure DNS</span><span class="sxs-lookup"><span data-stu-id="dd2ea-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="dd2ea-147">Atualize o ficheiro da zona DNS para o domínio.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="dd2ea-148">A Azure AD verificará a propriedade do nome de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="dd2ea-149">Utilize [o Azure DNS](/azure/dns/dns-getstarted-portal) para registos DNS Azure/Microsoft 365/externos em Azure, ou adicione a entrada de DNS num registo [DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="dd2ea-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="dd2ea-150">Registe um domínio personalizado com um registo público.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="dd2ea-151">Inicie sessão na entidade de registo de nome de domínio para o domínio.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="dd2ea-152">Um administrador aprovado pode ser necessário para fazer atualizações dns.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="dd2ea-153">Atualize o ficheiro da zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="dd2ea-154">(A entrada de DNS não afetará o encaminhamento de e-mail ou comportamentos de hospedagem web.)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="dd2ea-155">Crie uma aplicação web multi-nól padrão no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="dd2ea-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="dd2ea-156">Confule a integração contínua híbrida e a implementação contínua (CI/CD) para implementar aplicações web para Azure e Azure Stack Hub e para alterar automaticamente ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="dd2ea-157">Azure Stack Hub com imagens adequadas sincronizadas para executar (Windows Server e SQL) e implementação do Serviço de Aplicações são necessárias.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="dd2ea-158">Para mais informações, reveja a documentação do Serviço de Aplicações [Pré-requisitos para a implementação do Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="dd2ea-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="dd2ea-159">Adicionar código a Azure Repos</span><span class="sxs-lookup"><span data-stu-id="dd2ea-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="dd2ea-160">Repositórios do Azure</span><span class="sxs-lookup"><span data-stu-id="dd2ea-160">Azure Repos</span></span>

1. <span data-ttu-id="dd2ea-161">Inscreva-se no Azure Repos com uma conta que tem direitos de criação de projetos em Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="dd2ea-162">O CI/CD híbrido pode aplicar-se tanto ao código de aplicação como ao código de infraestrutura.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="dd2ea-163">Use [modelos de Gestor de Recursos Azure](https://azure.microsoft.com/resources/templates/) para o desenvolvimento de nuvem privada e hospedada.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Ligue-se a um projeto em Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="dd2ea-165">**Clone o repositório** criando e abrindo a aplicação web padrão.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clone repo em app web Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="dd2ea-167">Criar implementação de aplicativos web autossuficientes para Serviços de Aplicações em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="dd2ea-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="dd2ea-168">Editar o ficheiro **WebApplication.csproj.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="dd2ea-169">Selecione `Runtimeidentifier` e adicione `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="dd2ea-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="dd2ea-170">(Ver [documentação de implantação independente.)](/dotnet/core/deploying/deploy-with-vs#simpleSelf)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Editar o ficheiro do projeto de aplicativo web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="dd2ea-172">Consulte o código para Azure Repos usando o Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="dd2ea-173">Confirme que o código da aplicação foi verificado no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="dd2ea-174">Criar a definição de construção</span><span class="sxs-lookup"><span data-stu-id="dd2ea-174">Create the build definition</span></span>

1. <span data-ttu-id="dd2ea-175">Inscreva-se nos Pipelines Azure para confirmar a capacidade de criar definições de construção.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="dd2ea-176">Adicione **-r ganhar código 10-x64.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="dd2ea-177">Esta adição é necessária para desencadear uma implantação autossuficiente com .NET Core.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Adicione código à aplicação web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="dd2ea-179">Executar a construção.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-179">Run the build.</span></span> <span data-ttu-id="dd2ea-180">O processo [de construção de implantação independente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefactos que funcionam no Azure e no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="dd2ea-181">Use um agente azure hospedado</span><span class="sxs-lookup"><span data-stu-id="dd2ea-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="dd2ea-182">Usar um agente de construção hospedado em Azure Pipelines é uma opção conveniente para construir e implementar aplicações web.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="dd2ea-183">A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="dd2ea-184">Gerir e configurar o processo de CD</span><span class="sxs-lookup"><span data-stu-id="dd2ea-184">Manage and configure the CD process</span></span>

<span data-ttu-id="dd2ea-185">A Azure Pipelines e a Azure DevOps Services fornecem um oleoduto altamente configurável e manejável para libertações para vários ambientes como desenvolvimento, encenação, QA e ambientes de produção; incluindo a necessidade de aprovações em fases específicas.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="dd2ea-186">Criar definição de libertação</span><span class="sxs-lookup"><span data-stu-id="dd2ea-186">Create release definition</span></span>

1. <span data-ttu-id="dd2ea-187">Selecione o botão **mais** para adicionar uma nova versão no separador **Versões** na secção **'Construir e Soltar'** serviços Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Criar uma definição de versão](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="dd2ea-189">Aplique o modelo de implementação do serviço de aplicação da Azure.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-189">Apply the Azure App Service Deployment template.</span></span>

   ![Aplicar o modelo de implementação do serviço de aplicação da Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="dd2ea-191">Under **Add artifact**, adicione o artefacto para a aplicação de construção Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Adicione artefacto à construção da Nuvem Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="dd2ea-193">No separador Pipeline, selecione a **Fase,** Ligação de tarefa do ambiente e desave os valores do ambiente em nuvem Azure.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Definir valores do ambiente em nuvem Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="dd2ea-195">Desaponte o **nome do ambiente** e selecione a **subscrição Azure** para o ponto final da Nuvem Azure.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selecione a subscrição do Azure para o ponto final da Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="dd2ea-197">No **nome do serviço app,** desaprote o nome de serviço de aplicação Azure necessário.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Definir nome de serviço de aplicativo Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="dd2ea-199">Insira "Hosted VS2017" na **fila do Agente** para o ambiente hospedado na nuvem Azure.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Definir a fila do agente para o ambiente hospedado na nuvem Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="dd2ea-201">No menu Implementar o Serviço de Aplicações Azure, selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="dd2ea-202">Selecione **OK** para a **localização da pasta**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-202">Select **OK** to **folder location**.</span></span>
  
      ![Selecione pacote ou pasta para ambiente de Serviço de Aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selecione pacote ou pasta para ambiente de Serviço de Aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="dd2ea-205">Guarde todas as alterações e volte a **lançar o gasoduto**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Guardar alterações no gasoduto de libertação](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="dd2ea-207">Adicione um novo artefacto selecionando a construção para a aplicação Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Adicione novo artefacto para a aplicação Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="dd2ea-209">Adicione mais um ambiente aplicando a Implementação do Serviço de Aplicações Azure.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Adicionar ambiente à implementação do Serviço de Aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="dd2ea-211">Nomeie o novo ambiente "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="dd2ea-211">Name the new environment "Azure Stack".</span></span>

    ![Ambiente de nome na implementação do serviço de aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="dd2ea-213">Encontre o ambiente Azure Stack no **separador Tarefa.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Ambiente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="dd2ea-215">Selecione a subscrição para o ponto final Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Selecione a subscrição para o ponto final Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="dd2ea-217">Desaprote o nome da aplicação web Azure Stack como o nome do serviço de aplicação da App.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="dd2ea-218">![Definir nome de aplicativo web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="dd2ea-219">Selecione o agente Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-219">Select the Azure Stack agent.</span></span>

    ![Selecione o agente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="dd2ea-221">Na secção 'Implementar serviço de aplicações Azure', selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="dd2ea-222">Selecione **OK** para a localização da pasta.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-222">Select **OK** to folder location.</span></span>

    ![Selecione pasta para implementação do serviço de aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selecione pasta para implementação do serviço de aplicações Azure](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="dd2ea-225">No separador Variável adicione uma variável `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` nomeada, definir o seu valor como **verdadeiro**, e âmbito para Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Adicionar variável à implementação da app Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="dd2ea-227">Selecione o ícone de disparo de implementação **contínua** em ambos os artefactos e ative o gatilho de implementação **Continua.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selecione o gatilho de implementação contínua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="dd2ea-229">Selecione o ícone de condições **de pré-implantação** no ambiente Azure Stack e desaccione o gatilho para **depois de ser lançado.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Selecione condições de pré-implantação](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="dd2ea-231">Guarde todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="dd2ea-232">Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis ambientais](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição de libertação a partir de um modelo.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="dd2ea-233">Estas definições não podem ser modificadas nas definições de tarefa; em vez disso, o item ambiente parental deve ser selecionado para editar estas definições.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="dd2ea-234">Publicar no Azure Stack Hub via Visual Studio</span><span class="sxs-lookup"><span data-stu-id="dd2ea-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="dd2ea-235">Ao criar pontos finais, uma construção de Serviços Azure DevOps pode implementar aplicações do Serviço Azure para o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="dd2ea-236">A Azure Pipelines conecta-se ao agente de construção, que se conecta ao Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="dd2ea-237">Inscreva-se nos Serviços Azure DevOps e vá à página de definições da aplicação.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="dd2ea-238">Nas **Definições**, selecione **Security**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="dd2ea-239">Nos **grupos VSTS,** selecione **Criadores de Pontos Finais**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="dd2ea-240">No separador **Membros,** **selecione Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="dd2ea-241">In **Adicionar utilizadores e grupos,** introduza um nome de utilizador e selecione esse utilizador na lista de utilizadores.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="dd2ea-242">Selecione **Guardar alterações**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="dd2ea-243">Na lista de **grupos VSTS,** selecione **Administradores de pontos finais**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="dd2ea-244">No separador **Membros,** **selecione Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="dd2ea-245">In **Adicionar utilizadores e grupos,** introduza um nome de utilizador e selecione esse utilizador na lista de utilizadores.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="dd2ea-246">Selecione **Guardar alterações**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-246">Select **Save changes**.</span></span>

<span data-ttu-id="dd2ea-247">Agora que a informação do ponto final existe, a ligação Azure Pipelines para Azure Stack Hub está pronta a ser utilizada.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="dd2ea-248">O agente de construção em Azure Stack Hub recebe instruções da Azure Pipelines e, em seguida, o agente transmite informações de ponto final para comunicação com o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="dd2ea-249">Desenvolver a construção de aplicativos</span><span class="sxs-lookup"><span data-stu-id="dd2ea-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="dd2ea-250">Azure Stack Hub com imagens adequadas sincronizadas para executar (Windows Server e SQL) e implementação do Serviço de Aplicações são necessárias.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="dd2ea-251">Para obter mais informações, consulte [Pré-requisitos para a implementação do Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="dd2ea-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="dd2ea-252">Utilize [modelos de Gestor de Recursos Azure](https://azure.microsoft.com/resources/templates/) como código de aplicação web do Azure Repos para implementar em ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="dd2ea-253">Adicione código a um projeto Azure Repos</span><span class="sxs-lookup"><span data-stu-id="dd2ea-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="dd2ea-254">Inscreva-se no Azure Repos com uma conta que tem direitos de criação de projeto no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="dd2ea-255">**Clone o repositório** criando e abrindo a aplicação web padrão.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="dd2ea-256">Criar implementação de aplicativos web autossuficientes para Serviços de Aplicações em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="dd2ea-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="dd2ea-257">Editar o ficheiro **WebApplication.csproj:** Selecione `Runtimeidentifier` e adicione `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="dd2ea-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="dd2ea-258">Para obter mais informações, consulte a documentação [de implantação autónoma.](/dotnet/core/deploying/deploy-with-vs#simpleSelf)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="dd2ea-259">Utilize o Team Explorer para verificar o código em Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="dd2ea-260">Confirme que o código da aplicação foi verificado no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="dd2ea-261">Criar a definição de construção</span><span class="sxs-lookup"><span data-stu-id="dd2ea-261">Create the build definition</span></span>

1. <span data-ttu-id="dd2ea-262">Inscreva-se nos Pipelines Azure com uma conta que pode criar uma definição de construção.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="dd2ea-263">Aceda à página **Build Web Application** para o projeto.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="dd2ea-264">Em **Arguments**, add **-r win10-x64** código.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="dd2ea-265">Esta adição é necessária para desencadear uma implantação independente com .NET Core.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="dd2ea-266">Executar a construção.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-266">Run the build.</span></span> <span data-ttu-id="dd2ea-267">O processo [de construção de implantação independente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefactos que podem ser executados no Azure e no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="dd2ea-268">Use um agente de construção hospedado em Azure</span><span class="sxs-lookup"><span data-stu-id="dd2ea-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="dd2ea-269">Usar um agente de construção hospedado em Azure Pipelines é uma opção conveniente para construir e implementar aplicações web.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="dd2ea-270">A manutenção e as atualizações são feitas automaticamente pelo Microsoft Azure, permitindo um ciclo de desenvolvimento contínuo e ininterrupto.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="dd2ea-271">Configure o processo de implantação contínua (CD)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="dd2ea-272">A Azure Pipelines e a Azure DevOps Services fornecem um oleoduto altamente configurável e manejável para lançamentos para vários ambientes como desenvolvimento, encenação, garantia de qualidade (QA) e produção.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="dd2ea-273">Este processo pode incluir a necessidade de aprovações em fases específicas do ciclo de vida da aplicação.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="dd2ea-274">Criar definição de libertação</span><span class="sxs-lookup"><span data-stu-id="dd2ea-274">Create release definition</span></span>

<span data-ttu-id="dd2ea-275">Criar uma definição de lançamento é o passo final no processo de construção de aplicações.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="dd2ea-276">Esta definição de libertação é usada para criar uma libertação e implantar uma construção.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="dd2ea-277">Inscreva-se na Azure Pipelines e vá para **Construir e Lançar** para o projeto.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="dd2ea-278">No separador **Versões,** selecione **[ + ]** e, em seguida, escolha Criar **definição de desbloqueio**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="dd2ea-279">No **Selecionar um Modelo**, escolha a **implementação do serviço de aplicações Azure**e, em seguida, selecione **Apply**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="dd2ea-280">No **Add artifact**, a partir da **Definição Origem (Definição build)**, selecione a aplicação de construção Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="dd2ea-281">No **separador Pipeline,** selecione a ligação **1 Fase**, **1 Para** **Ver as tarefas ambientais**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="dd2ea-282">No separador **Tarefas, insira** O Azure como **o nome Ambiente** e selecione o EP AzureCloud Traders-Web da lista de **subscrições Azure.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="dd2ea-283">Introduza o nome de **serviço de aplicações Azure**, que está `northwindtraders` na próxima captura do ecrã.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="dd2ea-284">Para a fase agente, **selecione Hosted VS2017** da lista de **filas do Agente.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="dd2ea-285">No **Serviço de Aplicações Deploy Azure**, selecione o **Pacote ou pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="dd2ea-286">No **Ficheiro ou Pasta Select**, selecione **OK** para **localização**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="dd2ea-287">Guarde todas as alterações e volte ao **Pipeline.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="dd2ea-288">No **separador Pipeline,** **selecione Adicionar artefacto,** e escolha o **NorthwindCloud Traders-Vessel** da lista **Origem (Definição de Construção).**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="dd2ea-289">No **Selecionar um Modelo,** adicione outro ambiente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="dd2ea-290">Escolha **a aplicação do serviço de aplicações Azure** e, em seguida, selecione **Apply**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="dd2ea-291">Insira `Azure Stack Hub` como o nome **ambiente**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="dd2ea-292">No separador **Tarefas,** encontre e selecione O Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="dd2ea-293">A partir da lista **de subscrições do Azure,** selecione **AzureStack Traders-Vessel EP** para o ponto final do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="dd2ea-294">Insira o nome da aplicação web Azure Stack Hub como o nome do **serviço app.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="dd2ea-295">Sob **a seleção de agente,** escolha **AzureStack -b Douglas Fir** da lista de filas do **Agente.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="dd2ea-296">Para **implementar o Serviço de Aplicações Azure**, selecione o Pacote ou **pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="dd2ea-297">No **Ficheiro ou Na Pasta Select**, selecione **OK** para a **localização**da pasta .</span><span class="sxs-lookup"><span data-stu-id="dd2ea-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="dd2ea-298">No **separador Variável,** encontre a variável `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` nomeada.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="dd2ea-299">Desa estale o valor variável para **verdadeiro,** e desa esta medida para **o Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="dd2ea-300">No **separador Pipeline,** selecione o ícone **de disparo de implementação contínua** para o artefacto NorthwindCloud Traders-Web e desembra o gatilho de **implementação contínua** para **Ativado**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="dd2ea-301">Faça o mesmo com o artefacto **northwindCloud traders-vessel.**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="dd2ea-302">Para o ambiente Azure Stack Hub, selecione o ícone **de condições de pré-implantação** para definir o gatilho para **depois do lançamento**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="dd2ea-303">Guarde todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="dd2ea-304">Algumas definições para tarefas de libertação são definidas automaticamente como [variáveis ambientais](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição de libertação a partir de um modelo.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="dd2ea-305">Estas definições não podem ser modificadas nas definições de tarefa, mas podem ser modificadas nos itens do ambiente dos pais.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="dd2ea-306">Criar um lançamento</span><span class="sxs-lookup"><span data-stu-id="dd2ea-306">Create a release</span></span>

1. <span data-ttu-id="dd2ea-307">No **separador Pipeline,** abra a lista **de Desbloqueio** e selecione **Criar verte**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="dd2ea-308">Introduza uma descrição para a libertação, verifique se os artefactos corretos estão selecionados e, em seguida, selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="dd2ea-309">Após alguns momentos, aparece um banner indicando que o novo lançamento foi criado e o nome de lançamento é apresentado como um link.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="dd2ea-310">Selecione o link para ver a página do resumo do lançamento.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="dd2ea-311">A página do resumo do lançamento mostra detalhes sobre o lançamento.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="dd2ea-312">Na seguinte captura de ecrã para "Release-2", a secção **Ambiente** mostra o **estado de Implementação** do Azure como "EM PROGRESSO", e o estado de Azure Stack Hub é "SUCCEEDED".</span><span class="sxs-lookup"><span data-stu-id="dd2ea-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="dd2ea-313">Quando o estado de implantação do ambiente Azure muda para "SUCCEEDED", aparece um banner indicando que o desbloqueio está pronto para aprovação.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="dd2ea-314">Quando uma implantação está pendente ou falhou, é mostrado um ícone de informação azul **(i).**</span><span class="sxs-lookup"><span data-stu-id="dd2ea-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="dd2ea-315">Passe por cima do ícone para ver um pop-up que contém o motivo do atraso ou da falha.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="dd2ea-316">Outras vistas, como a lista de lançamentos, também exibem um ícone que indica que a aprovação está pendente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="dd2ea-317">O pop-up para este ícone mostra o nome do ambiente e mais detalhes relacionados com a implementação.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="dd2ea-318">É fácil para um administrador ver o progresso geral dos lançamentos e ver que lançamentos estão à espera de aprovação.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="dd2ea-319">Monitorização e implantação de faixas</span><span class="sxs-lookup"><span data-stu-id="dd2ea-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="dd2ea-320">Na página de resumo **Do Lançamento-2,** selecione **Registos**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="dd2ea-321">Durante uma implementação, esta página mostra o registo ao vivo do agente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="dd2ea-322">O painel esquerdo mostra o estado de cada operação na implantação para cada ambiente.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="dd2ea-323">Selecione o ícone da pessoa na coluna **Ação** para uma aprovação pré-implantação ou pós-implantação para ver quem aprovou (ou rejeitou) a implementação e a mensagem que forneceram.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="dd2ea-324">Após o acabamento da colocação, todo o ficheiro de registo é apresentado no painel direito.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="dd2ea-325">Selecione qualquer **passo** no painel esquerdo para ver o ficheiro de registo para um único passo, como **Inicialize Job**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="dd2ea-326">A capacidade de ver troncos individuais facilita o rastreio e depuração de partes da implantação geral.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="dd2ea-327">**Guarde** o ficheiro de registo para um passo ou **Descarregue todos os registos como zip**.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="dd2ea-328">Abra o **separador Resumo** para ver informações gerais sobre o lançamento.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="dd2ea-329">Esta visão mostra detalhes sobre a construção, os ambientes para os quais foi implantado, estado de implantação e outras informações sobre a libertação.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="dd2ea-330">Selecione uma ligação ambiental **(Azure** ou **Azure Stack Hub**) para ver informações sobre as implementações existentes e pendentes para um ambiente específico.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="dd2ea-331">Use estas vistas como uma forma rápida de verificar se a mesma construção foi implantada em ambos os ambientes.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="dd2ea-332">Abra a **aplicação de produção implantada** num browser.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="dd2ea-333">Por exemplo, para o website Azure App Services, abra o URL `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="dd2ea-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="dd2ea-334">Integração do Azure e do Azure Stack Hub proporciona uma solução de nuvem transversal escalável</span><span class="sxs-lookup"><span data-stu-id="dd2ea-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="dd2ea-335">Um serviço multi-nuvem flexível e robusto proporciona segurança de dados, back up e redundância, disponibilidade consistente e rápida, armazenamento e distribuição escaláveis e encaminhamento geocompatível.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="dd2ea-336">Este processo desencadeado manualmente garante uma troca de carga fiável e eficiente entre aplicações web hospedadas e disponibilidade imediata de dados cruciais.</span><span class="sxs-lookup"><span data-stu-id="dd2ea-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="dd2ea-337">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="dd2ea-337">Next steps</span></span>

- <span data-ttu-id="dd2ea-338">Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)</span><span class="sxs-lookup"><span data-stu-id="dd2ea-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
