---
title: Implementar aplicativo híbrido com dados no local que escalam a nuvem cruzada
description: Saiba como implementar uma aplicação que utiliza dados no local e escala em nuvem cruzada usando Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353483"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="3ab84-103">Implementar aplicativo híbrido com dados no local que escalam a nuvem cruzada</span><span class="sxs-lookup"><span data-stu-id="3ab84-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="3ab84-104">Este guia de solução mostra-lhe como implementar uma aplicação híbrida que abrange tanto o Azure como o Azure Stack Hub e utiliza uma única fonte de dados no local.</span><span class="sxs-lookup"><span data-stu-id="3ab84-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="3ab84-105">Ao utilizar uma solução de nuvem híbrida, pode combinar os benefícios de conformidade de uma nuvem privada com a escalabilidade da nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="3ab84-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="3ab84-106">Os seus desenvolvedores também podem aproveitar o ecossistema de desenvolvedores da Microsoft e aplicar as suas habilidades nos ambientes de nuvem e no local.</span><span class="sxs-lookup"><span data-stu-id="3ab84-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="3ab84-107">Visão geral e pressupostos</span><span class="sxs-lookup"><span data-stu-id="3ab84-107">Overview and assumptions</span></span>

<span data-ttu-id="3ab84-108">Siga este tutorial para configurar um fluxo de trabalho que permite aos desenvolvedores implementar uma aplicação web idêntica para uma nuvem pública e uma nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="3ab84-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="3ab84-109">Esta aplicação pode aceder a uma rede de encaminhamento não internet hospedada na nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="3ab84-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="3ab84-110">Estas aplicações web são monitorizadas e quando há um pico no tráfego, um programa modifica os registos DNS para redirecionar o tráfego para a nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="3ab84-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="3ab84-111">Quando o tráfego cai para o nível antes do pico, o tráfego é encaminhado de volta para a nuvem privada.</span><span class="sxs-lookup"><span data-stu-id="3ab84-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="3ab84-112">Este tutorial abrange as seguintes tarefas:</span><span class="sxs-lookup"><span data-stu-id="3ab84-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3ab84-113">Implemente um servidor de base de dados SQL Server ligado a híbridos.</span><span class="sxs-lookup"><span data-stu-id="3ab84-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="3ab84-114">Ligue uma aplicação web no Azure global a uma rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="3ab84-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="3ab84-115">Configure DNS para escalar as nuvens cruzadas.</span><span class="sxs-lookup"><span data-stu-id="3ab84-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="3ab84-116">Configure os certificados SSL para a escala de nuvens cruzadas.</span><span class="sxs-lookup"><span data-stu-id="3ab84-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="3ab84-117">Configure e implemente a aplicação web.</span><span class="sxs-lookup"><span data-stu-id="3ab84-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="3ab84-118">Crie um perfil de Gestor de Tráfego e configuure-o para a escala de nuvens cruzadas.</span><span class="sxs-lookup"><span data-stu-id="3ab84-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="3ab84-119">Configurar a monitorização e alerta de Informações de Aplicação para o aumento do tráfego.</span><span class="sxs-lookup"><span data-stu-id="3ab84-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="3ab84-120">Configure a troca automática de tráfego entre o Azure global e o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="3ab84-121">![Diagrama de pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3ab84-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3ab84-122">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3ab84-123">O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="3ab84-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3ab84-124">O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas.</span><span class="sxs-lookup"><span data-stu-id="3ab84-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3ab84-125">As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="3ab84-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="3ab84-126">Pressupostos</span><span class="sxs-lookup"><span data-stu-id="3ab84-126">Assumptions</span></span>

<span data-ttu-id="3ab84-127">Este tutorial pressupõe que você tem um conhecimento básico de Azure global e Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="3ab84-128">Se quiser saber mais antes de iniciar o tutorial, reveja estes artigos:</span><span class="sxs-lookup"><span data-stu-id="3ab84-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="3ab84-129">Introdução ao Azure</span><span class="sxs-lookup"><span data-stu-id="3ab84-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="3ab84-130">Conceitos de chave de hub de pilha de Azure</span><span class="sxs-lookup"><span data-stu-id="3ab84-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="3ab84-131">Este tutorial também assume que tem uma subscrição do Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="3ab84-132">Se não tiver uma subscrição, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="3ab84-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3ab84-133">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="3ab84-133">Prerequisites</span></span>

<span data-ttu-id="3ab84-134">Antes de iniciar esta solução, certifique-se de que cumpre os seguintes requisitos:</span><span class="sxs-lookup"><span data-stu-id="3ab84-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="3ab84-135">Um Kit de Desenvolvimento de Pilhas Azure (ASDK) ou uma subscrição de um Sistema Integrado Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="3ab84-136">Para implementar o ASDK, siga as instruções em [Implementar o ASDK utilizando o instalador](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="3ab84-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="3ab84-137">A sua instalação Azure Stack Hub deve ter o seguinte instalado:</span><span class="sxs-lookup"><span data-stu-id="3ab84-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="3ab84-138">O Serviço de Aplicações Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-138">The Azure App Service.</span></span> <span data-ttu-id="3ab84-139">Trabalhe com o seu Operador Azure Stack Hub para implementar e configurar o Serviço de Aplicações Azure no seu ambiente.</span><span class="sxs-lookup"><span data-stu-id="3ab84-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="3ab84-140">Este tutorial requer que o Serviço de Aplicações tenha pelo menos um (1) papel dedicado ao trabalhador disponível.</span><span class="sxs-lookup"><span data-stu-id="3ab84-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="3ab84-141">Uma imagem do Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="3ab84-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="3ab84-142">Um Windows Server 2016 com uma imagem do Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="3ab84-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="3ab84-143">Os planos e ofertas apropriados.</span><span class="sxs-lookup"><span data-stu-id="3ab84-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="3ab84-144">Um nome de domínio para a sua aplicação web.</span><span class="sxs-lookup"><span data-stu-id="3ab84-144">A domain name for your web app.</span></span> <span data-ttu-id="3ab84-145">Se não tiver um nome de domínio, pode comprar um a um fornecedor de domínio como GoDaddy, Bluehost e InMotion.</span><span class="sxs-lookup"><span data-stu-id="3ab84-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="3ab84-146">Um certificado SSL para o seu domínio a partir de uma autoridade de certificado de confiança como o LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="3ab84-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="3ab84-147">Uma aplicação web que comunica com uma base de dados do SQL Server e suporta o Application Insights.</span><span class="sxs-lookup"><span data-stu-id="3ab84-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="3ab84-148">Você pode baixar a aplicação de amostra [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) do GitHub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="3ab84-149">Uma rede híbrida entre uma rede virtual Azure e a rede virtual Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="3ab84-150">Para obter instruções detalhadas, consulte [a conectividade em nuvem híbrida Configure com O Azure e o Azure Stack Hub](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="3ab84-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="3ab84-151">Um gasoduto híbrido de integração contínua/implantação contínua (CI/CD) com um agente de construção privada no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="3ab84-152">Para obter instruções detalhadas, consulte [a identidade em nuvem híbrida Configure com aplicações Azure Stack Hub](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="3ab84-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="3ab84-153">Implementar um servidor de base de dados sql ligado híbrido</span><span class="sxs-lookup"><span data-stu-id="3ab84-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="3ab84-154">Assine no portal de utilizadores Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="3ab84-155">No **Painel de Instrumentos**, selecione **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Mercado Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="3ab84-157">No **Marketplace**, selecione **Compute**, e depois escolha **Mais**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="3ab84-158">Em **Mais**, selecione a **Licença de servidor SQL grátis: SQL Server 2017 Developer na** imagem do Servidor do Windows.</span><span class="sxs-lookup"><span data-stu-id="3ab84-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Selecione uma imagem de máquina virtual no portal de utilizador Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="3ab84-160">Na **Licença de servidor SQL grátis: SQL Server 2017 Developer on Windows Server**, selecione **Create**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="3ab84-161">Em **Basics > configurações básicas**de configuração , forneça um **Nome** para a máquina virtual (VM), um nome **de utilizador** para o SQL Server SA e uma **Palavra-passe** para a SA.</span><span class="sxs-lookup"><span data-stu-id="3ab84-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="3ab84-162">A partir da lista de drop-down de **subscrição,** selecione a subscrição para a qual está a implementar.</span><span class="sxs-lookup"><span data-stu-id="3ab84-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="3ab84-163">Para **o grupo de recursos,** utilize escolha o VM **existente** e coloque o VM no mesmo grupo de recursos que a sua aplicação web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Configurar definições básicas para VM no portal de utilizador Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="3ab84-165">Under **Size,** escolha um tamanho para o seu VM.</span><span class="sxs-lookup"><span data-stu-id="3ab84-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="3ab84-166">Para este tutorial, recomendamos A2_Standard ou um DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="3ab84-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="3ab84-167">Em **Definições > configurar funcionalidades opcionais,** configurar as seguintes definições:</span><span class="sxs-lookup"><span data-stu-id="3ab84-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="3ab84-168">**Conta de armazenamento**: Crie uma nova conta se precisar de uma.</span><span class="sxs-lookup"><span data-stu-id="3ab84-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="3ab84-169">**Rede virtual:**</span><span class="sxs-lookup"><span data-stu-id="3ab84-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="3ab84-170">Certifique-se de que o seu SQL Server VM está implantado na mesma rede virtual que os gateways VPN.</span><span class="sxs-lookup"><span data-stu-id="3ab84-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="3ab84-171">**Endereço IP público**: Utilize as definições predefinidos.</span><span class="sxs-lookup"><span data-stu-id="3ab84-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="3ab84-172">**Grupo de segurança de rede**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="3ab84-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="3ab84-173">Criar um novo NSG.</span><span class="sxs-lookup"><span data-stu-id="3ab84-173">Create a new NSG.</span></span>
   - <span data-ttu-id="3ab84-174">**Extensões e Monitorização**: Mantenha as definições predefinidos.</span><span class="sxs-lookup"><span data-stu-id="3ab84-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="3ab84-175">**Conta de armazenamento de**diagnóstico: Crie uma nova conta se precisar de uma.</span><span class="sxs-lookup"><span data-stu-id="3ab84-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="3ab84-176">Selecione **OK** para guardar a sua configuração.</span><span class="sxs-lookup"><span data-stu-id="3ab84-176">Select **OK** to save your configuration.</span></span>

     ![Configurar funcionalidades opcionais de VM no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="3ab84-178">Nas **definições do SQL Server,** configure as seguintes definições:</span><span class="sxs-lookup"><span data-stu-id="3ab84-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="3ab84-179">Para **conectividade SQL,** selecione **Public (Internet)**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="3ab84-180">Para **o Porto**, mantenha o padrão, **1433**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="3ab84-181">Para **autenticação SQL**, selecione **Ativar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="3ab84-182">Quando ativa a autenticação SQL, deve povoar-se automaticamente com a informação "SQLAdmin" que configuraste em **Básicos.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="3ab84-183">Para o resto das definições, mantenha as predefinições.</span><span class="sxs-lookup"><span data-stu-id="3ab84-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="3ab84-184">Selecione **OK**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-184">Select **OK**.</span></span>

     ![Configurar as definições do Servidor SQL no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="3ab84-186">Em **Resumo,** reveja a configuração VM e, em seguida, selecione **OK** para iniciar a implementação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Resumo de configuração no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="3ab84-188">Leva algum tempo para criar o novo VM.</span><span class="sxs-lookup"><span data-stu-id="3ab84-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="3ab84-189">Pode ver o estado dos seus VMs em **máquinas Virtuais.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Estado das máquinas virtuais no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="3ab84-191">Criar aplicativos web em Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3ab84-192">O Azure App Service simplifica a execução e gestão de uma aplicação web.</span><span class="sxs-lookup"><span data-stu-id="3ab84-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="3ab84-193">Como o Azure Stack Hub é consistente com o Azure, o Serviço de Aplicações pode funcionar em ambos os ambientes.</span><span class="sxs-lookup"><span data-stu-id="3ab84-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="3ab84-194">Utilizará o Serviço de Aplicações para hospedar a sua aplicação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="3ab84-195">Criar aplicativos web</span><span class="sxs-lookup"><span data-stu-id="3ab84-195">Create web apps</span></span>

1. <span data-ttu-id="3ab84-196">Crie uma aplicação web em Azure seguindo as instruções em [Gerir um plano de Serviço de Aplicações em Azure.](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)</span><span class="sxs-lookup"><span data-stu-id="3ab84-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="3ab84-197">Certifique-se de colocar a aplicação web no mesmo grupo de subscrição e recursos que a sua rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="3ab84-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="3ab84-198">Repita o passo anterior (1) no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="3ab84-199">Adicione rota para Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="3ab84-200">O Serviço de Aplicações no Azure Stack Hub deve ser redirecionável a partir da internet pública para permitir que os utilizadores acedam à sua aplicação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="3ab84-201">Se o seu Azure Stack Hub estiver acessível a partir da internet, tome nota do endereço IP ou URL virado para o público para a aplicação web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="3ab84-202">Se estiver a usar um ASDK, pode [configurar um mapeamento ESTÁTICO NAT](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) para expor o Serviço de Aplicações fora do ambiente virtual.</span><span class="sxs-lookup"><span data-stu-id="3ab84-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="3ab84-203">Conecte uma aplicação web em Azure a uma rede híbrida</span><span class="sxs-lookup"><span data-stu-id="3ab84-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="3ab84-204">Para fornecer conectividade entre a extremidade frontal da web em Azure e a base de dados SQL Server em Azure Stack Hub, a aplicação web deve ser conectada à rede híbrida entre Azure e Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="3ab84-205">Para ativar a conectividade, terá de:</span><span class="sxs-lookup"><span data-stu-id="3ab84-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="3ab84-206">Configurar a conectividade ponto-a-local.</span><span class="sxs-lookup"><span data-stu-id="3ab84-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="3ab84-207">Configure a aplicação web.</span><span class="sxs-lookup"><span data-stu-id="3ab84-207">Configure the web app.</span></span>
- <span data-ttu-id="3ab84-208">Modifique o portal de rede local no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="3ab84-209">Configurar a rede virtual Azure para a conectividade ponto-a-local</span><span class="sxs-lookup"><span data-stu-id="3ab84-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="3ab84-210">O gateway de rede virtual no lado Azure da rede híbrida deve permitir que as ligações ponto-a-local se integrem com o Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="3ab84-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="3ab84-211">No portal Azure, aceda à página de gateway de rede virtual.</span><span class="sxs-lookup"><span data-stu-id="3ab84-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="3ab84-212">Em **Definições**, selecione **configuração ponto-a-local**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Opção ponto-a-local no gateway de rede virtual Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="3ab84-214">Selecione **Configurar agora** para configurar ponto a local.</span><span class="sxs-lookup"><span data-stu-id="3ab84-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Iniciar configuração ponto-a-local no gateway de rede virtual Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="3ab84-216">Na página de configuração **ponto-a-local,** insira o intervalo de endereço IP privado que pretende utilizar na **piscina Address**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="3ab84-217">Certifique-se de que o intervalo especificado não se sobrepõe a nenhuma das gamas de endereços já utilizadas pelas sub-redes nos componentes globais do Azure ou do Azure Stack Hub da rede híbrida.</span><span class="sxs-lookup"><span data-stu-id="3ab84-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="3ab84-218">Sob **o tipo de túnel,** desmarque a **VPN IKEv2**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="3ab84-219">**Selecione Guardar** para terminar a configuração ponto a local.</span><span class="sxs-lookup"><span data-stu-id="3ab84-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Definições ponto-a-local no gateway de rede virtual Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="3ab84-221">Integrar a app Azure App Service com a rede híbrida</span><span class="sxs-lookup"><span data-stu-id="3ab84-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="3ab84-222">Para ligar a aplicação ao Azure VNet, siga as instruções em Gateway necessárias para [a integração do VNet](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="3ab84-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="3ab84-223">Vá a **Definições** para o plano de Serviço de Aplicações que hospeda a aplicação web.</span><span class="sxs-lookup"><span data-stu-id="3ab84-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="3ab84-224">Em **Definições**, selecione **Networking**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-224">In **Settings**, select **Networking**.</span></span>

    ![Configurar rede para o plano de Serviço de Aplicações](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="3ab84-226">Na **Integração VNET,** selecione **Clique aqui para gerir.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Gerir a integração do VNET para o plano de Serviço de Aplicações](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="3ab84-228">Selecione o VNET que pretende configurar.</span><span class="sxs-lookup"><span data-stu-id="3ab84-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="3ab84-229">Nos **endereços IP ROUTED TO VNET,** insira o intervalo de endereços IP para o Azure VNet, o Azure Stack Hub VNet e os espaços de endereço ponto a local.</span><span class="sxs-lookup"><span data-stu-id="3ab84-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="3ab84-230">**Selecione Guardar** para validar e guardar estas definições.</span><span class="sxs-lookup"><span data-stu-id="3ab84-230">Select **Save** to validate and save these settings.</span></span>

    ![Intervalos de endereços IP para rota na Integração de Rede Virtual](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="3ab84-232">Para saber mais sobre como o Serviço de Aplicações se integra com os VNets Azure, consulte [Integrar a sua aplicação com uma Rede Virtual Azure.](/azure/app-service/web-sites-integrate-with-vnet)</span><span class="sxs-lookup"><span data-stu-id="3ab84-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="3ab84-233">Configure a rede virtual Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="3ab84-234">O portal de rede local na rede virtual Azure Stack Hub precisa de ser configurado para encaminhar o tráfego a partir da gama de endereços ponto-a-local do Serviço de Aplicações.</span><span class="sxs-lookup"><span data-stu-id="3ab84-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="3ab84-235">No portal Azure Stack Hub, aceda ao **gateway de rede local.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-235">In the Azure Stack Hub portal, go to **Local network gateway**.</span></span> <span data-ttu-id="3ab84-236">Em **Definições**, selecione **Configuração**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-236">Under **Settings**, select **Configuration**.</span></span>

    ![Opção de configuração gateway em Azure Stack Hub gateway de rede local](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="3ab84-238">No **espaço Address**, insira a gama de endereços ponto-a-local para o gateway de rede virtual em Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Espaço de endereço ponto-a-local no gateway de rede local Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="3ab84-240">**Selecione Guardar** para validar e guardar a configuração.</span><span class="sxs-lookup"><span data-stu-id="3ab84-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="3ab84-241">Configurar DNS para escala de nuvem cruzada</span><span class="sxs-lookup"><span data-stu-id="3ab84-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="3ab84-242">Ao configurar adequadamente o DNS para aplicações cross-cloud, os utilizadores podem aceder às instâncias globais do Azure Stack Hub da sua aplicação web.</span><span class="sxs-lookup"><span data-stu-id="3ab84-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="3ab84-243">A configuração do DNS para este tutorial também permite que o Azure Traffic Manager encaminhe o tráfego quando a carga aumenta ou diminui.</span><span class="sxs-lookup"><span data-stu-id="3ab84-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="3ab84-244">Este tutorial utiliza o Azure DNS para gerir o DNS porque os domínios do Serviço de Aplicações não funcionam.</span><span class="sxs-lookup"><span data-stu-id="3ab84-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="3ab84-245">Criar subdomínios</span><span class="sxs-lookup"><span data-stu-id="3ab84-245">Create subdomains</span></span>

<span data-ttu-id="3ab84-246">Como o Traffic Manager depende de DNS CNAMEs, é necessário um subdomínio para encaminhar adequadamente o tráfego para os pontos finais.</span><span class="sxs-lookup"><span data-stu-id="3ab84-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="3ab84-247">Para obter mais informações sobre registos DNS e mapeamento de domínio, consulte [os domínios do mapa com o Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="3ab84-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="3ab84-248">Para o ponto final do Azure, irá criar um subdomínio que os utilizadores podem usar para aceder à sua aplicação web.</span><span class="sxs-lookup"><span data-stu-id="3ab84-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="3ab84-249">Para este tutorial, pode utilizar **app.northwind.com,** mas deve personalizar este valor com base no seu próprio domínio.</span><span class="sxs-lookup"><span data-stu-id="3ab84-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="3ab84-250">Também terás de criar um subdomínio com um disco A para o ponto final do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="3ab84-251">Pode **usá azurestack.northwind.com.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="3ab84-252">Configure um domínio personalizado em Azure</span><span class="sxs-lookup"><span data-stu-id="3ab84-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="3ab84-253">Adicione o **nome de anfitrião app.northwind.com** à aplicação web Azure [mapeando um CNAME ao Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="3ab84-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="3ab84-254">Configure domínios personalizados no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="3ab84-255">Adicione o **nome de anfitrião azurestack.northwind.com** à aplicação web Azure Stack Hub [mapeando um disco A ao Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="3ab84-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="3ab84-256">Utilize o endereço IP de encaminhamento de internet para a aplicação Do Serviço de Aplicações.</span><span class="sxs-lookup"><span data-stu-id="3ab84-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="3ab84-257">Adicione o **nome de anfitrião app.northwind.com** à aplicação web Azure Stack Hub [mapeando um CNAME para O Serviço de Aplicações Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="3ab84-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="3ab84-258">Utilize o nome de anfitrião configurado no passo anterior (1) como alvo para o CNAME.</span><span class="sxs-lookup"><span data-stu-id="3ab84-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="3ab84-259">Configure certificados SSL para escalamento de nuvens cruzadas</span><span class="sxs-lookup"><span data-stu-id="3ab84-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="3ab84-260">É importante garantir que os dados sensíveis recolhidos pela sua aplicação web são seguros em trânsito para e quando armazenados na base de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="3ab84-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="3ab84-261">Irá configurar as suas aplicações web Azure e Azure Stack Hub para utilizar certificados SSL para todo o tráfego de entrada.</span><span class="sxs-lookup"><span data-stu-id="3ab84-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="3ab84-262">Adicione SSL ao Azure e ao Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3ab84-263">Para adicionar SSL a Azure:</span><span class="sxs-lookup"><span data-stu-id="3ab84-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="3ab84-264">Certifique-se de que o certificado SSL que obtém é válido para o subdomínio que criou.</span><span class="sxs-lookup"><span data-stu-id="3ab84-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="3ab84-265">(Não faz mal usar certificados wildcard.)</span><span class="sxs-lookup"><span data-stu-id="3ab84-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="3ab84-266">No portal Azure, siga as instruções na **aplicação Web** e ligue as secções de **certificado SSL** do [Bind um certificado SSL personalizado existente ao artigo da Azure Web Apps.](/azure/app-service/app-service-web-tutorial-custom-ssl)</span><span class="sxs-lookup"><span data-stu-id="3ab84-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="3ab84-267">Selecione **SSL baseado em SNI** como o **Tipo SSL**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="3ab84-268">Redirecione todo o tráfego para a porta HTTPS.</span><span class="sxs-lookup"><span data-stu-id="3ab84-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="3ab84-269">Siga as instruções na secção   **HttpS da Secção HTTPS** do [Vinco um certificado SSL personalizado existente para](/azure/app-service/app-service-web-tutorial-custom-ssl) o artigo da Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="3ab84-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="3ab84-270">Para adicionar SSL ao Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="3ab84-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="3ab84-271">Repita os passos 1-3 que usou para o Azure, utilizando o portal Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="3ab84-272">Configurar e implementar a aplicação Web</span><span class="sxs-lookup"><span data-stu-id="3ab84-272">Configure and deploy the web app</span></span>

<span data-ttu-id="3ab84-273">Irá configurar o código da aplicação para reportar telemetria à instância correta do Application Insights e configurar as aplicações web com as cadeias de conexão certas.</span><span class="sxs-lookup"><span data-stu-id="3ab84-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="3ab84-274">Para saber mais sobre a Aplicação Insights, veja [o que é Insights de Aplicação?](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="3ab84-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="3ab84-275">Adicionar Insights de Aplicação</span><span class="sxs-lookup"><span data-stu-id="3ab84-275">Add Application Insights</span></span>

1. <span data-ttu-id="3ab84-276">Abra a sua aplicação web no Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="3ab84-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="3ab84-277">[Adicione Insights de Aplicação](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) ao seu projeto para transmitir a telemetria que o Application Insights utiliza para criar alertas quando o tráfego web aumenta ou diminui.</span><span class="sxs-lookup"><span data-stu-id="3ab84-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="3ab84-278">Configurar cordas de ligação dinâmica</span><span class="sxs-lookup"><span data-stu-id="3ab84-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="3ab84-279">Cada instância da aplicação web usará um método diferente para ligar à base de dados SQL.</span><span class="sxs-lookup"><span data-stu-id="3ab84-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="3ab84-280">A aplicação em Azure utiliza o endereço IP privado do SQL Server VM e a aplicação no Azure Stack Hub utiliza o endereço IP público do SQL Server VM.</span><span class="sxs-lookup"><span data-stu-id="3ab84-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="3ab84-281">Num sistema integrado Azure Stack Hub, o endereço IP público não deve ser dirijo.</span><span class="sxs-lookup"><span data-stu-id="3ab84-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="3ab84-282">Numa ASDK, o endereço IP público não é roteado fora da ASDK.</span><span class="sxs-lookup"><span data-stu-id="3ab84-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="3ab84-283">Pode utilizar variáveis ambientais do Serviço de Aplicações para passar uma cadeia de ligação diferente a cada instância da aplicação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="3ab84-284">Abra a aplicação no Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="3ab84-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="3ab84-285">Abra Startup.cs e encontre o seguinte bloco de código:</span><span class="sxs-lookup"><span data-stu-id="3ab84-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="3ab84-286">Substitua o bloco de código anterior pelo seguinte código, que utiliza uma cadeia de ligação definida no *appsettings.jsno* ficheiro:</span><span class="sxs-lookup"><span data-stu-id="3ab84-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="3ab84-287">Configurar configurar as configurações da aplicação do Serviço de Aplicações de Aplicações</span><span class="sxs-lookup"><span data-stu-id="3ab84-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="3ab84-288">Crie cordas de conexão para Azure e Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="3ab84-289">As cordas devem ser as mesmas, exceto os endereços IP que são utilizados.</span><span class="sxs-lookup"><span data-stu-id="3ab84-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="3ab84-290">No Azure e no Azure Stack Hub, adicione a cadeia de conexão apropriada [como uma definição](/azure/app-service/web-sites-configure) de aplicação na aplicação web, usando `SQLCONNSTR\_` como um prefixo no nome.</span><span class="sxs-lookup"><span data-stu-id="3ab84-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="3ab84-291">**Guarde** as definições da aplicação web e reinicie a aplicação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="3ab84-292">Permitir a escala automática no Azure global</span><span class="sxs-lookup"><span data-stu-id="3ab84-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="3ab84-293">Quando cria a sua aplicação web num ambiente de Serviço de Aplicações, começa com um exemplo.</span><span class="sxs-lookup"><span data-stu-id="3ab84-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="3ab84-294">Pode escalar automaticamente para adicionar instâncias para fornecer mais recursos computacional para a sua aplicação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="3ab84-295">Da mesma forma, pode escalar automaticamente e reduzir o número de casos de que a sua aplicação necessita.</span><span class="sxs-lookup"><span data-stu-id="3ab84-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="3ab84-296">Precisa de ter um plano de Serviço de Aplicações para configurar escala e escalar.</span><span class="sxs-lookup"><span data-stu-id="3ab84-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="3ab84-297">Se não tem um plano, crie um antes de começar os próximos passos.</span><span class="sxs-lookup"><span data-stu-id="3ab84-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="3ab84-298">Permitir a escala automática</span><span class="sxs-lookup"><span data-stu-id="3ab84-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="3ab84-299">No portal Azure, encontre o plano de Serviço de Aplicações para os sites que pretende escalar e, em seguida, selecione **Scale-out (Plano de Serviço de Aplicações)**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Dimensione o Serviço de Aplicações Azure](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="3ab84-301">**Selecione Ative autoscale**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-301">Select **Enable autoscale**.</span></span>

    ![Permitir a autoescalação no Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="3ab84-303">Introduza um nome para **Nome de Definição de Escala Automática**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="3ab84-304">Para a regra de escala automática **predefinido,** selecione **Escala com base numa métrica**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="3ab84-305">Definir os **limites de instância** para **mínimo: 1,** **Máximo: 10**, e **Predefinido: 1**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Configurar autoescalação no Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="3ab84-307">**Selecione +Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="3ab84-308">Na **Fonte Métrica**, selecione **O Recurso Atual**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="3ab84-309">Utilize os seguintes critérios e ações para a regra.</span><span class="sxs-lookup"><span data-stu-id="3ab84-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="3ab84-310">Critérios</span><span class="sxs-lookup"><span data-stu-id="3ab84-310">Criteria</span></span>

1. <span data-ttu-id="3ab84-311">Em **Agregação de Tempo,** selecione **Média**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="3ab84-312">Em **Nome Métrico**, selecione **PERCENTAGEM CPU**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="3ab84-313">No **Operador**, selecione **Maior que**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="3ab84-314">Definir o **Limiar** para **50**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="3ab84-315">Definir a **Duração** para **10**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="3ab84-316">Ação</span><span class="sxs-lookup"><span data-stu-id="3ab84-316">Action</span></span>

1. <span data-ttu-id="3ab84-317">Em **operação**, selecione **Aumentar a Contagem por**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="3ab84-318">Definir o **Conde de Instância** para **2**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="3ab84-319">**Desafrie** o arrefecimento para **5**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="3ab84-320">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-320">Select **Add**.</span></span>

5. <span data-ttu-id="3ab84-321">Selecione o **+ Adicionar uma regra**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="3ab84-322">Na **Fonte Métrica,** selecione **O Recurso Atual.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="3ab84-323">O recurso atual conterá o nome/GUID do seu plano de Serviço de Aplicações e as listas de drop-down tipo de **recurso** e **recursos** não estarão disponíveis.</span><span class="sxs-lookup"><span data-stu-id="3ab84-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="3ab84-324">Ativar a escala automática em</span><span class="sxs-lookup"><span data-stu-id="3ab84-324">Enable automatic scale in</span></span>

<span data-ttu-id="3ab84-325">Quando o tráfego diminui, a aplicação web Azure pode reduzir automaticamente o número de casos ativos para reduzir custos.</span><span class="sxs-lookup"><span data-stu-id="3ab84-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="3ab84-326">Esta ação é menos agressiva do que a escala e minimiza o impacto nos utilizadores de aplicações.</span><span class="sxs-lookup"><span data-stu-id="3ab84-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="3ab84-327">Vá para a condição de escala **padrão** e, em seguida, **selecione + Adicione uma regra**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="3ab84-328">Utilize os seguintes critérios e ações para a regra.</span><span class="sxs-lookup"><span data-stu-id="3ab84-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="3ab84-329">Critérios</span><span class="sxs-lookup"><span data-stu-id="3ab84-329">Criteria</span></span>

1. <span data-ttu-id="3ab84-330">Em **Agregação de Tempo,** selecione **Média**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="3ab84-331">Em **Nome Métrico**, selecione **PERCENTAGEM CPU**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="3ab84-332">No **Operador**, selecione **Menos de**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="3ab84-333">Definir o **Limiar** para **30**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="3ab84-334">Definir a **Duração** para **10**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="3ab84-335">Ação</span><span class="sxs-lookup"><span data-stu-id="3ab84-335">Action</span></span>

1. <span data-ttu-id="3ab84-336">Em **operação**, **selecione Decrease Count by**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="3ab84-337">Decrete o **Conde de Instância** para **1**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="3ab84-338">**Desafrie** o arrefecimento para **5**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="3ab84-339">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="3ab84-340">Crie um perfil de Gestor de Tráfego e configuure a escala de nuvens cruzadas</span><span class="sxs-lookup"><span data-stu-id="3ab84-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="3ab84-341">Crie um perfil de Gestor de Tráfego utilizando o portal Azure e, em seguida, configuure pontos finais para permitir a escala de nuvens cruzadas.</span><span class="sxs-lookup"><span data-stu-id="3ab84-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="3ab84-342">Criar perfil de Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="3ab84-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="3ab84-343">Selecione **Criar um recurso**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="3ab84-344">Selecione **Rede**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-344">Select **Networking**.</span></span>
3. <span data-ttu-id="3ab84-345">Selecione **o perfil do Gestor de Tráfego** e configufique as seguintes definições:</span><span class="sxs-lookup"><span data-stu-id="3ab84-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="3ab84-346">Em **Nome,** insira um nome para o seu perfil.</span><span class="sxs-lookup"><span data-stu-id="3ab84-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="3ab84-347">Este nome **deve** ser único na zona trafficmanager.net e é usado para criar um novo nome DNS (por exemplo, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="3ab84-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="3ab84-348">Para **o método de encaminhamento,** selecione o **Ponderado**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="3ab84-349">Para **Subscrição**, selecione a subscrição em que pretende criar este perfil.</span><span class="sxs-lookup"><span data-stu-id="3ab84-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="3ab84-350">No **Grupo de Recursos,** crie um novo grupo de recursos para este perfil.</span><span class="sxs-lookup"><span data-stu-id="3ab84-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="3ab84-351">Em **Localização do grupo de recursos**, selecione a localização do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="3ab84-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="3ab84-352">Esta definição refere-se à localização do grupo de recursos e não tem qualquer impacto no perfil do Gestor de Tráfego que é implantado globalmente.</span><span class="sxs-lookup"><span data-stu-id="3ab84-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="3ab84-353">Selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-353">Select **Create**.</span></span>

    ![Criar perfil de Gestor de Tráfego](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="3ab84-355">Quando a implementação global do seu perfil de Gestor de Tráfego está completa, é mostrada na lista de recursos para o grupo de recursos que o criou.</span><span class="sxs-lookup"><span data-stu-id="3ab84-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="3ab84-356">Adicionar pontos finais do Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="3ab84-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="3ab84-357">Procure o perfil de Gestor de Tráfego que criou.</span><span class="sxs-lookup"><span data-stu-id="3ab84-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="3ab84-358">Se navegou para o grupo de recursos para o perfil, selecione o perfil.</span><span class="sxs-lookup"><span data-stu-id="3ab84-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="3ab84-359">No **perfil de Gestor de Tráfego**, em **DEFINIÇÕES,** selecione **Pontos de final**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="3ab84-360">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-360">Select **Add**.</span></span>

4. <span data-ttu-id="3ab84-361">No **ponto final adicionar,** utilize as seguintes definições para Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="3ab84-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="3ab84-362">Para **o tipo**, selecione ponto final **externo**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="3ab84-363">Insira um **nome** para o ponto final.</span><span class="sxs-lookup"><span data-stu-id="3ab84-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="3ab84-364">Para **nome de domínio totalmente qualificado (FQDN) ou IP,** insira o URL externo para a sua aplicação web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="3ab84-365">Para **o peso**, mantenha o padrão, **1**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="3ab84-366">Este peso resulta em todo o tráfego indo para este ponto final se for saudável.</span><span class="sxs-lookup"><span data-stu-id="3ab84-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="3ab84-367">Deixe **adicionar como desativado** sem verificação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="3ab84-368">Selecione **OK** para guardar o ponto final do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="3ab84-369">Vai configurar o ponto final do Azure a seguir.</span><span class="sxs-lookup"><span data-stu-id="3ab84-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="3ab84-370">No **perfil de Gestor de Tráfego**, selecione **Pontos de final**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="3ab84-371">Selecione **+Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-371">Select **+Add**.</span></span>
3. <span data-ttu-id="3ab84-372">No **ponto final adicionar,** utilize as seguintes definições para Azure:</span><span class="sxs-lookup"><span data-stu-id="3ab84-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="3ab84-373">Para **tipo**, selecione **Azure endpoint**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="3ab84-374">Insira um **nome** para o ponto final.</span><span class="sxs-lookup"><span data-stu-id="3ab84-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="3ab84-375">Para **o tipo de recurso Alvo**, selecione App **Service**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="3ab84-376">Para **o recurso Target**, **selecione Escolha um serviço** de aplicações para ver uma lista de Aplicações Web na mesma subscrição.</span><span class="sxs-lookup"><span data-stu-id="3ab84-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="3ab84-377">Em **Recurso**, escolha o Serviço de Aplicações que pretende adicionar como o primeiro ponto final.</span><span class="sxs-lookup"><span data-stu-id="3ab84-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="3ab84-378">Para **peso**, selecione **2**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="3ab84-379">Esta definição resulta em todo o tráfego que vai para este ponto final se o ponto final principal não for saudável, ou se tiver uma regra/alerta que redirecione o tráfego quando ativado.</span><span class="sxs-lookup"><span data-stu-id="3ab84-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="3ab84-380">Deixe **adicionar como desativado** sem verificação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="3ab84-381">Selecione **OK** para guardar o ponto final do Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="3ab84-382">Depois de configurados ambos os pontos finais, estão listados no **perfil de Gestor de Tráfego** quando seleciona **Pontos de Final**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="3ab84-383">O exemplo na seguinte captura de ecrã mostra dois pontos finais, com informações de estado e de configuração para cada um.</span><span class="sxs-lookup"><span data-stu-id="3ab84-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Pontos finais no perfil do Gestor de Tráfego](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="3ab84-385">Configurar a monitorização e alerta de Insights de Aplicação em Azure</span><span class="sxs-lookup"><span data-stu-id="3ab84-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="3ab84-386">O Azure Application Insights permite-lhe monitorizar a sua aplicação e enviar alertas com base nas condições que configura.</span><span class="sxs-lookup"><span data-stu-id="3ab84-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="3ab84-387">Alguns exemplos são: a aplicação está indisponível, está a sofrer falhas ou está a apresentar problemas de desempenho.</span><span class="sxs-lookup"><span data-stu-id="3ab84-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="3ab84-388">Utilizará as métricas Azure Application Insights para criar alertas.</span><span class="sxs-lookup"><span data-stu-id="3ab84-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="3ab84-389">Quando estes alertas disparam, a instância da sua aplicação web mudará automaticamente de Azure Stack Hub para Azure para escalar e, em seguida, de volta para Azure Stack Hub para escalar dentro</span><span class="sxs-lookup"><span data-stu-id="3ab84-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="3ab84-390">Criar um alerta a partir de métricas</span><span class="sxs-lookup"><span data-stu-id="3ab84-390">Create an alert from metrics</span></span>

<span data-ttu-id="3ab84-391">No portal Azure, vá ao grupo de recursos para este tutorial e selecione a instância Application Insights para abrir **Insights de Aplicação**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="3ab84-393">Você usará esta vista para criar um alerta de escala e um alerta de escala.</span><span class="sxs-lookup"><span data-stu-id="3ab84-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="3ab84-394">Crie o alerta de escala</span><span class="sxs-lookup"><span data-stu-id="3ab84-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="3ab84-395">Em **CONFIGURE,** **selecione Alertas (clássicos)**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="3ab84-396">**Selecione Adicionar alerta métrico (clássico)**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="3ab84-397">Na **regra adicionar,** configurar as seguintes definições:</span><span class="sxs-lookup"><span data-stu-id="3ab84-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="3ab84-398">Para **nome**, **insira Burst into Azure Cloud**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="3ab84-399">Uma **descrição** é opcional.</span><span class="sxs-lookup"><span data-stu-id="3ab84-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="3ab84-400">Em Alerta **de Fonte**  >  **em**, selecione **Métricas**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="3ab84-401">De **Acordo com critérios,** selecione a sua subscrição, o grupo de recursos para o seu perfil de Gestor de Tráfego e o nome do perfil de Gestor de Tráfego para o recurso.</span><span class="sxs-lookup"><span data-stu-id="3ab84-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="3ab84-402">Para **métrica**, selecione **Taxa de Pedido**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="3ab84-403">Para **a condição**, selecione Maior **que**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="3ab84-404">Para **limiar**, insira **2**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="3ab84-405">Para **o período**, selecione Ao longo dos **últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="3ab84-406">Em **notificação através de:**</span><span class="sxs-lookup"><span data-stu-id="3ab84-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="3ab84-407">Verifique a caixa de verificação para **os proprietários, colaboradores e leitores de e-mail.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="3ab84-408">Insira o seu endereço de e-mail para **e-mails adicionais do administrador.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="3ab84-409">Na barra de menus, **selecione Save**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="3ab84-410">Crie o alerta de escala</span><span class="sxs-lookup"><span data-stu-id="3ab84-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="3ab84-411">Em **CONFIGURE,** **selecione Alertas (clássicos)**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="3ab84-412">**Selecione Adicionar alerta métrico (clássico)**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="3ab84-413">Na **regra adicionar,** configurar as seguintes definições:</span><span class="sxs-lookup"><span data-stu-id="3ab84-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="3ab84-414">Para **nome**, **introduza a Escala de volta no Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="3ab84-415">Uma **descrição** é opcional.</span><span class="sxs-lookup"><span data-stu-id="3ab84-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="3ab84-416">Em Alerta **de Fonte**  >  **em**, selecione **Métricas**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="3ab84-417">De **Acordo com critérios,** selecione a sua subscrição, o grupo de recursos para o seu perfil de Gestor de Tráfego e o nome do perfil de Gestor de Tráfego para o recurso.</span><span class="sxs-lookup"><span data-stu-id="3ab84-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="3ab84-418">Para **métrica**, selecione **Taxa de Pedido**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="3ab84-419">Para **a condição**, selecione Menos **de**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="3ab84-420">Para **limiar**, insira **2**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="3ab84-421">Para **o período**, selecione Ao longo dos **últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="3ab84-422">Em **notificação através de:**</span><span class="sxs-lookup"><span data-stu-id="3ab84-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="3ab84-423">Verifique a caixa de verificação para **os proprietários, colaboradores e leitores de e-mail.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="3ab84-424">Insira o seu endereço de e-mail para **e-mails adicionais do administrador.**</span><span class="sxs-lookup"><span data-stu-id="3ab84-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="3ab84-425">Na barra de menus, **selecione Save**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="3ab84-426">A imagem que se segue mostra os alertas para a escala e a escala.</span><span class="sxs-lookup"><span data-stu-id="3ab84-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Alertas de Insights de Aplicação (clássico)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="3ab84-428">Redirecione o tráfego entre Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3ab84-429">Pode configurar a comutação manual ou automática do tráfego da sua aplicação web entre o Azure e o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="3ab84-430">Configurar a troca manual entre O Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3ab84-431">Quando o seu site atingir os limiares que configura, receberá um alerta.</span><span class="sxs-lookup"><span data-stu-id="3ab84-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="3ab84-432">Utilize os seguintes passos para redirecionar manualmente o tráfego para Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="3ab84-433">No portal Azure, selecione o seu perfil de Gestor de Tráfego.</span><span class="sxs-lookup"><span data-stu-id="3ab84-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Pontos finais do Gestor de Tráfego no portal Azure](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="3ab84-435">Selecione **Pontos de final**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="3ab84-436">Selecione o **ponto final Azure**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="3ab84-437">Em **Status**, selecione **Ativado**e, em seguida, selecione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Ativar o ponto final do Azure no portal Azure](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="3ab84-439">Em **Pontos de fim** para o perfil de Gestor de Tráfego, selecione Ponto final **Externo**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="3ab84-440">Em **Estado de Estado**, selecione **Desativado**e, em seguida, selecione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="3ab84-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Desativar o ponto final do Azure Stack Hub no portal Azure](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="3ab84-442">Depois de configurados os pontos finais, o tráfego de aplicações vai para a sua aplicação web de escala Azure em vez da aplicação web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Pontos finais alterados no tráfego de aplicações web Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="3ab84-444">Para inverter o fluxo de volta para Azure Stack Hub, use os passos anteriores para:</span><span class="sxs-lookup"><span data-stu-id="3ab84-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="3ab84-445">Ativar o ponto de terminação do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="3ab84-446">Desative o ponto final do Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="3ab84-447">Configurar a comutação automática entre O Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3ab84-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3ab84-448">Também pode utilizar a monitorização do Application Insights se a sua aplicação funcionar num ambiente [sem servidor](https://azure.microsoft.com/overview/serverless-computing/) fornecido pelas Funções Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="3ab84-449">Neste cenário, pode configurar o Application Insights para usar um webhook que chama uma aplicação de função.</span><span class="sxs-lookup"><span data-stu-id="3ab84-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="3ab84-450">Esta aplicação ativa ou desativa automaticamente um ponto final em resposta a um alerta.</span><span class="sxs-lookup"><span data-stu-id="3ab84-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="3ab84-451">Utilize os seguintes passos como guia para configurar a comutação automática de tráfego.</span><span class="sxs-lookup"><span data-stu-id="3ab84-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="3ab84-452">Crie uma aplicação Azure Function.</span><span class="sxs-lookup"><span data-stu-id="3ab84-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="3ab84-453">Crie uma função acionada por HTTP.</span><span class="sxs-lookup"><span data-stu-id="3ab84-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="3ab84-454">Importe os Azure SDKs para Gestor de Recursos, Web Apps e Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="3ab84-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="3ab84-455">Desenvolver código para:</span><span class="sxs-lookup"><span data-stu-id="3ab84-455">Develop code to:</span></span>

   - <span data-ttu-id="3ab84-456">Autenticar a sua assinatura Azure.</span><span class="sxs-lookup"><span data-stu-id="3ab84-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="3ab84-457">Utilize um parâmetro que alterne os pontos finais do Gestor de Tráfego para direcionar o tráfego para o Azure ou para o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3ab84-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="3ab84-458">Guarde o seu código e adicione o URL da aplicação de função com os parâmetros apropriados à secção **Webhook** das definições de regra de alerta de Insights de Aplicação.</span><span class="sxs-lookup"><span data-stu-id="3ab84-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="3ab84-459">O tráfego é redirecionado automaticamente quando um alerta de Insights de Aplicação dispara.</span><span class="sxs-lookup"><span data-stu-id="3ab84-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3ab84-460">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="3ab84-460">Next steps</span></span>

- <span data-ttu-id="3ab84-461">Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)</span><span class="sxs-lookup"><span data-stu-id="3ab84-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
