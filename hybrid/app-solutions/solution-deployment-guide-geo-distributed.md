---
title: Tráfego direto com uma app geo-distribuída usando Azure e Azure Stack Hub
description: Saiba como direcionar o tráfego para pontos finais específicos com uma solução de aplicações geo-distribuídas utilizando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 741ddf2c3ed234788af359dd233f6a656fbea13c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477359"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="7064e-103">Tráfego direto com uma app geo-distribuída usando Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="7064e-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="7064e-104">Aprenda a direcionar o tráfego para pontos finais específicos com base em várias métricas usando o padrão de aplicações geo-distribuídas.</span><span class="sxs-lookup"><span data-stu-id="7064e-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="7064e-105">A criação de um perfil de Gestor de Tráfego com configuração geográfica de encaminhamento e ponto final garante que a informação é encaminhada para pontos finais com base nos requisitos regionais, na regulação corporativa e internacional e nas necessidades dos seus dados.</span><span class="sxs-lookup"><span data-stu-id="7064e-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="7064e-106">Nesta solução, você construirá um ambiente de amostra para:</span><span class="sxs-lookup"><span data-stu-id="7064e-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="7064e-107">Crie uma aplicação geo-distribuída.</span><span class="sxs-lookup"><span data-stu-id="7064e-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="7064e-108">Use o Traffic Manager para direcionar a sua aplicação.</span><span class="sxs-lookup"><span data-stu-id="7064e-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="7064e-109">Use o padrão de aplicações geo-distribuídas</span><span class="sxs-lookup"><span data-stu-id="7064e-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="7064e-110">Com o padrão geo-distribuído, a sua aplicação abrange regiões.</span><span class="sxs-lookup"><span data-stu-id="7064e-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="7064e-111">Pode falhar na nuvem pública, mas alguns dos seus utilizadores podem exigir que os seus dados permaneçam na sua região.</span><span class="sxs-lookup"><span data-stu-id="7064e-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="7064e-112">Pode direcionar os utilizadores para a nuvem mais adequada com base nas suas necessidades.</span><span class="sxs-lookup"><span data-stu-id="7064e-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="7064e-113">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="7064e-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="7064e-114">Considerações de escalabilidade</span><span class="sxs-lookup"><span data-stu-id="7064e-114">Scalability considerations</span></span>

<span data-ttu-id="7064e-115">A solução que vai construir com este artigo não é para acomodar a escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="7064e-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="7064e-116">No entanto, se for utilizado em combinação com outras soluções Azure e no local, pode acomodar requisitos de escalabilidade.</span><span class="sxs-lookup"><span data-stu-id="7064e-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="7064e-117">Para obter informações sobre a criação de uma solução híbrida com autoscaling através do gestor de tráfego, consulte [Criar soluções de escala cross-cloud com o Azure.](solution-deployment-guide-cross-cloud-scaling.md)</span><span class="sxs-lookup"><span data-stu-id="7064e-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="7064e-118">Considerações de disponibilidade</span><span class="sxs-lookup"><span data-stu-id="7064e-118">Availability considerations</span></span>

<span data-ttu-id="7064e-119">Como é o caso das considerações de escalabilidade, esta solução não aborda diretamente a disponibilidade.</span><span class="sxs-lookup"><span data-stu-id="7064e-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="7064e-120">No entanto, a Azure e as soluções no local podem ser implementadas nesta solução para garantir uma elevada disponibilidade para todos os componentes envolvidos.</span><span class="sxs-lookup"><span data-stu-id="7064e-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="7064e-121">Quando utilizar este padrão</span><span class="sxs-lookup"><span data-stu-id="7064e-121">When to use this pattern</span></span>

- <span data-ttu-id="7064e-122">A sua organização tem sucursais internacionais que exigem políticas de segurança e distribuição regionais personalizadas.</span><span class="sxs-lookup"><span data-stu-id="7064e-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="7064e-123">Cada um dos escritórios da sua organização retira dados de funcionários, negócios e instalações, o que requer atividade de reporte de acordo com os regulamentos locais e fusos horários.</span><span class="sxs-lookup"><span data-stu-id="7064e-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="7064e-124">Os requisitos de alta escala são cumpridos através da escala horizontal de aplicações com múltiplas implementações de aplicações dentro de uma única região e em todas as regiões para lidar com requisitos de carga extrema.</span><span class="sxs-lookup"><span data-stu-id="7064e-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="7064e-125">Planejando a topologia</span><span class="sxs-lookup"><span data-stu-id="7064e-125">Planning the topology</span></span>

<span data-ttu-id="7064e-126">Antes de construir uma pegada de aplicativo distribuída, ajuda a saber as seguintes coisas:</span><span class="sxs-lookup"><span data-stu-id="7064e-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="7064e-127">**Domínio personalizado para a aplicação:** Qual é o nome de domínio personalizado que os clientes usarão para aceder à aplicação?</span><span class="sxs-lookup"><span data-stu-id="7064e-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="7064e-128">Para a aplicação da amostra, o nome de domínio personalizado é *www \. scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="7064e-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="7064e-129">**Domínio do Gestor de Tráfego:** Um nome de domínio é escolhido ao criar um [perfil de Gestor de Tráfego Azure](/azure/traffic-manager/traffic-manager-manage-profiles).</span><span class="sxs-lookup"><span data-stu-id="7064e-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="7064e-130">Este nome é combinado com o *sufixo* trafficmanager.net para registar uma entrada de domínio que é gerida pelo Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="7064e-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="7064e-131">Para a aplicação da amostra, o nome escolhido é *escalável-ase-demo*.</span><span class="sxs-lookup"><span data-stu-id="7064e-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="7064e-132">Como resultado, o nome de domínio completo que é gerido pelo Traffic Manager é *scalable-ase-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="7064e-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="7064e-133">**Estratégia para escalar a pegada da aplicação:** Decida se a pegada da aplicação será distribuída por vários ambientes do Serviço de Aplicações numa única região, várias regiões ou uma mistura de ambas as abordagens.</span><span class="sxs-lookup"><span data-stu-id="7064e-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="7064e-134">A decisão deve basear-se nas expectativas de onde o tráfego de clientes terá origem e em que ponto o resto da infraestrutura de back-end suportada de uma aplicação pode escalar.</span><span class="sxs-lookup"><span data-stu-id="7064e-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="7064e-135">Por exemplo, com uma aplicação 100% apátrida, uma aplicação pode ser massivamente dimensionada usando uma combinação de múltiplos ambientes de Serviço de Aplicações por região de Azure, multiplicados por ambientes de Serviço de Aplicações implantados em várias regiões do Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="7064e-136">Com mais de 15 regiões globais de Azure disponíveis para escolher, os clientes podem realmente construir uma pegada de aplicações de hiperescala em todo o mundo.</span><span class="sxs-lookup"><span data-stu-id="7064e-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="7064e-137">Para a aplicação de amostras aqui utilizada, três ambientes de Serviço de Aplicações foram criados numa única região de Azure (South Central US).</span><span class="sxs-lookup"><span data-stu-id="7064e-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="7064e-138">**Convenção de nomeação para os ambientes do Serviço de Aplicações:** Cada ambiente de Serviço de Aplicações requer um nome único.</span><span class="sxs-lookup"><span data-stu-id="7064e-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="7064e-139">Além de um ou dois ambientes de Serviço de Aplicações, é útil ter uma convenção de nomeação para ajudar a identificar cada ambiente de Serviço de Aplicações.</span><span class="sxs-lookup"><span data-stu-id="7064e-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="7064e-140">Para a aplicação de amostras usada aqui, foi usada uma simples convenção de nomeação.</span><span class="sxs-lookup"><span data-stu-id="7064e-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="7064e-141">Os nomes dos três ambientes do Serviço de Aplicações são *fe1ase,* *fe2ase*e *fe3ase.*</span><span class="sxs-lookup"><span data-stu-id="7064e-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="7064e-142">**Convenção de nomeação para as aplicações:** Uma vez que serão implementados vários casos da aplicação, é necessário um nome para cada instância da aplicação implementada.</span><span class="sxs-lookup"><span data-stu-id="7064e-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="7064e-143">Com o App Service Environment para Aplicações de Energia, o mesmo nome de aplicação pode ser usado em vários ambientes.</span><span class="sxs-lookup"><span data-stu-id="7064e-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="7064e-144">Uma vez que cada ambiente do Serviço de Aplicações tem um sufixo de domínio único, os desenvolvedores podem optar por reutilizar exatamente o mesmo nome de aplicação em cada ambiente.</span><span class="sxs-lookup"><span data-stu-id="7064e-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="7064e-145">Por exemplo, um programador poderia ter apps nomeadas da seguinte forma: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, e assim por diante.</span><span class="sxs-lookup"><span data-stu-id="7064e-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="7064e-146">Para a aplicação usada aqui, cada instância de aplicação tem um nome único.</span><span class="sxs-lookup"><span data-stu-id="7064e-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="7064e-147">Os nomes de instâncias de aplicações utilizados são *webfrontend1,* *webfrontend2*e *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="7064e-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="7064e-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="7064e-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="7064e-149">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="7064e-150">O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="7064e-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="7064e-151">O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas.</span><span class="sxs-lookup"><span data-stu-id="7064e-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="7064e-152">As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="7064e-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="7064e-153">Parte 1: Criar uma aplicação geo-distribuída</span><span class="sxs-lookup"><span data-stu-id="7064e-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="7064e-154">Nesta parte, vai criar uma aplicação web.</span><span class="sxs-lookup"><span data-stu-id="7064e-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="7064e-155">Crie aplicativos web e publique.</span><span class="sxs-lookup"><span data-stu-id="7064e-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="7064e-156">Adicione código ao Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="7064e-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="7064e-157">Aponte a construção da aplicação para vários alvos em nuvem.</span><span class="sxs-lookup"><span data-stu-id="7064e-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="7064e-158">Gerir e configurar o processo de CD.</span><span class="sxs-lookup"><span data-stu-id="7064e-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="7064e-159">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="7064e-159">Prerequisites</span></span>

<span data-ttu-id="7064e-160">É necessária uma subscrição Azure e uma instalação do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="7064e-161">Passos de aplicação geo-distribuídos</span><span class="sxs-lookup"><span data-stu-id="7064e-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="7064e-162">Obtenha um domínio personalizado e configuure DNS</span><span class="sxs-lookup"><span data-stu-id="7064e-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="7064e-163">Atualize o ficheiro da zona DNS para o domínio.</span><span class="sxs-lookup"><span data-stu-id="7064e-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="7064e-164">A Azure AD pode então verificar a propriedade do nome de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="7064e-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="7064e-165">Utilize [o Azure DNS](/azure/dns/dns-getstarted-portal) para registos DNS Azure/Office 365/external DNS dentro do Azure, ou adicione a entrada de DNS [num registo DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="7064e-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="7064e-166">Registe um domínio personalizado com um registo público.</span><span class="sxs-lookup"><span data-stu-id="7064e-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="7064e-167">Inicie sessão na entidade de registo de nome de domínio para o domínio.</span><span class="sxs-lookup"><span data-stu-id="7064e-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="7064e-168">Um administrador aprovado pode ser necessário para escruissar as atualizações do DNS.</span><span class="sxs-lookup"><span data-stu-id="7064e-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="7064e-169">Atualize o ficheiro da zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="7064e-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="7064e-170">A entrada do DNS não altera comportamentos como o encaminhamento de correio ou o hospedagem na Web.</span><span class="sxs-lookup"><span data-stu-id="7064e-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="7064e-171">Criar aplicativos web e publicar</span><span class="sxs-lookup"><span data-stu-id="7064e-171">Create web apps and publish</span></span>

<span data-ttu-id="7064e-172">Confule a integração contínua híbrida/entrega contínua (CI/CD) para implementar a Web App para Azure e Azure Stack Hub, e empurre automaticamente as alterações em ambas as nuvens.</span><span class="sxs-lookup"><span data-stu-id="7064e-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="7064e-173">Azure Stack Hub com imagens adequadas sincronizadas para executar (Windows Server e SQL) e implementação do Serviço de Aplicações são necessárias.</span><span class="sxs-lookup"><span data-stu-id="7064e-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="7064e-174">Para obter mais informações, consulte [Pré-requisitos para a implementação do Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="7064e-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="7064e-175">Adicionar código a Azure Repos</span><span class="sxs-lookup"><span data-stu-id="7064e-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="7064e-176">Inscreva-se no Visual Studio com uma **conta que tem direitos de criação de projetos** no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="7064e-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="7064e-177">O CI/CD pode aplicar-se tanto ao código de aplicação como ao código de infraestrutura.</span><span class="sxs-lookup"><span data-stu-id="7064e-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="7064e-178">Use [modelos de Gestor de Recursos Azure](https://azure.microsoft.com/resources/templates/) para o desenvolvimento de nuvem privada e hospedada.</span><span class="sxs-lookup"><span data-stu-id="7064e-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Conecte-se a um projeto em Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="7064e-180">**Clone o repositório** criando e abrindo a aplicação web padrão.</span><span class="sxs-lookup"><span data-stu-id="7064e-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Repositório de clones no Estúdio Visual](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="7064e-182">Criar implementação de aplicativos web em ambas as nuvens</span><span class="sxs-lookup"><span data-stu-id="7064e-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="7064e-183">Editar o ficheiro **WebApplication.csproj:** Selecione `Runtimeidentifier` e adicione `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="7064e-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="7064e-184">(Ver [documentação de implantação independente.)](/dotnet/core/deploying/deploy-with-vs#simpleSelf)</span><span class="sxs-lookup"><span data-stu-id="7064e-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Editar o ficheiro do projeto de aplicativo web no Estúdio Visual](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="7064e-186">**Consulte o código para Azure Repos** usando o Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="7064e-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="7064e-187">Confirme que o código de **aplicação** foi verificado no Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="7064e-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="7064e-188">Criar a definição de construção</span><span class="sxs-lookup"><span data-stu-id="7064e-188">Create the build definition</span></span>

1. <span data-ttu-id="7064e-189">**Inscreva-se nos Pipelines Azure** para confirmar a capacidade de criar definições de construção.</span><span class="sxs-lookup"><span data-stu-id="7064e-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="7064e-190">Adicione `-r win10-x64` código.</span><span class="sxs-lookup"><span data-stu-id="7064e-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="7064e-191">Esta adição é necessária para desencadear uma implantação autossuficiente com .NET Core.</span><span class="sxs-lookup"><span data-stu-id="7064e-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Adicione código à definição de construção em Gasodutos Azure](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="7064e-193">**Executar a construção.**</span><span class="sxs-lookup"><span data-stu-id="7064e-193">**Run the build**.</span></span> <span data-ttu-id="7064e-194">O processo [de construção de implantação independente](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará artefactos que podem ser executados no Azure e no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="7064e-195">Usando um agente azure hospedado</span><span class="sxs-lookup"><span data-stu-id="7064e-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="7064e-196">Usar um agente hospedado em Azure Pipelines é uma opção conveniente para construir e implementar aplicações web.</span><span class="sxs-lookup"><span data-stu-id="7064e-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="7064e-197">A manutenção e as atualizações são executadas automaticamente pela Microsoft Azure, o que permite o desenvolvimento, teste e implementação ininterruptas.</span><span class="sxs-lookup"><span data-stu-id="7064e-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="7064e-198">Gerir e configurar o processo de CD</span><span class="sxs-lookup"><span data-stu-id="7064e-198">Manage and configure the CD process</span></span>

<span data-ttu-id="7064e-199">Os Serviços Azure DevOps fornecem um oleoduto altamente configurável e manejável para libertações para vários ambientes, tais como desenvolvimento, encenação, QA e ambientes de produção; incluindo a necessidade de aprovações em fases específicas.</span><span class="sxs-lookup"><span data-stu-id="7064e-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="7064e-200">Criar definição de libertação</span><span class="sxs-lookup"><span data-stu-id="7064e-200">Create release definition</span></span>

1. <span data-ttu-id="7064e-201">Selecione o botão **mais** para adicionar uma nova versão no separador **Versões** na secção **'Construir e Soltar'** serviços Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="7064e-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Criar uma definição de lançamento nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="7064e-203">Aplique o modelo de implementação do serviço de aplicação da Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-203">Apply the Azure App Service Deployment template.</span></span>

   ![Aplicar o modelo de implementação do serviço de aplicação da Azure em serviços de devops Azure](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="7064e-205">Under **Add artifact**, adicione o artefacto para a aplicação de construção Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="7064e-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Adicione artefacto à Azure Cloud build em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="7064e-207">No separador Pipeline, selecione a **Fase,** Ligação de tarefa do ambiente e desave os valores do ambiente em nuvem Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Definir valores de ambiente em nuvem Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="7064e-209">Desaponte o **nome do ambiente** e selecione a **subscrição Azure** para o ponto final da Nuvem Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selecione a subscrição Azure para o ponto final da Azure Cloud nos serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="7064e-211">No **nome do serviço app,** desaprote o nome de serviço de aplicação Azure necessário.</span><span class="sxs-lookup"><span data-stu-id="7064e-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Definir nome de serviço de aplicativo Azure em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="7064e-213">Insira "Hosted VS2017" na **fila do Agente** para o ambiente hospedado na nuvem Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Definir a fila do agente para o ambiente hospedado na nuvem Azure DevOps](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="7064e-215">No menu Implementar o Serviço de Aplicações Azure, selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="7064e-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="7064e-216">Selecione **OK** para a **localização da pasta**.</span><span class="sxs-lookup"><span data-stu-id="7064e-216">Select **OK** to **folder location**.</span></span>
  
      ![Selecione pacote ou pasta para o ambiente do Serviço de Aplicações Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selecione pacote ou pasta para o ambiente do Serviço de Aplicações Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="7064e-219">Guarde todas as alterações e volte a **lançar o gasoduto**.</span><span class="sxs-lookup"><span data-stu-id="7064e-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Economize alterações no oleoduto de lançamento nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="7064e-221">Adicione um novo artefacto selecionando a construção para a aplicação Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Adicione novo artefacto para app Azure Stack Hub em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="7064e-223">Adicione mais um ambiente aplicando a Implementação do Serviço de Aplicações Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Adicionar ambiente à implementação do Serviço de Aplicações Azure em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="7064e-225">Nomeie o novo ambiente Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-225">Name the new environment Azure Stack Hub.</span></span>

    ![Ambiente de nome na implementação do serviço de aplicações Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="7064e-227">Encontre o ambiente Azure Stack Hub no **separador Tarefa.**</span><span class="sxs-lookup"><span data-stu-id="7064e-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Ambiente Azure Stack Hub em Serviços Azure DevOps em Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="7064e-229">Selecione a subscrição para o ponto final do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Selecione a subscrição do ponto final do Azure Stack Hub nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="7064e-231">Desaprote o nome da aplicação web Azure Stack Hub como o nome do serviço de aplicações da App.</span><span class="sxs-lookup"><span data-stu-id="7064e-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Definir nome de aplicativo web Azure Stack Hub em Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="7064e-233">Selecione o agente Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-233">Select the Azure Stack Hub agent.</span></span>

    ![Selecione o agente Azure Stack Hub nos serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="7064e-235">Na secção 'Implementar serviço de aplicações Azure', selecione o **Pacote ou Pasta** válido para o ambiente.</span><span class="sxs-lookup"><span data-stu-id="7064e-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="7064e-236">Selecione **OK** para a localização da pasta.</span><span class="sxs-lookup"><span data-stu-id="7064e-236">Select **OK** to folder location.</span></span>

    ![Selecione pasta para implantação do serviço de aplicações Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selecione pasta para implantação do serviço de aplicações Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="7064e-239">No separador Variável adicione uma variável `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` nomeada, definir o seu valor como **verdadeiro**, e âmbito para Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Adicionar variável à implementação de aplicativos Azure em serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="7064e-241">Selecione o ícone de disparo de implementação **contínua** em ambos os artefactos e ative o gatilho de implementação **Continua.**</span><span class="sxs-lookup"><span data-stu-id="7064e-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selecione o gatilho de implementação contínua nos serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="7064e-243">Selecione o ícone de condições **de pré-implantação** no ambiente Azure Stack Hub e desaccione o gatilho para **depois do lançamento.**</span><span class="sxs-lookup"><span data-stu-id="7064e-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Selecione condições de pré-implantação nos Serviços Azure DevOps](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="7064e-245">Guarde todas as alterações.</span><span class="sxs-lookup"><span data-stu-id="7064e-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="7064e-246">Algumas configurações para as tarefas podem ter sido definidas automaticamente como [variáveis ambientais](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) ao criar uma definição de libertação a partir de um modelo.</span><span class="sxs-lookup"><span data-stu-id="7064e-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="7064e-247">Estas definições não podem ser modificadas nas definições de tarefa; em vez disso, o item ambiente parental deve ser selecionado para editar estas definições.</span><span class="sxs-lookup"><span data-stu-id="7064e-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="7064e-248">Parte 2: Atualizar as opções de aplicações web</span><span class="sxs-lookup"><span data-stu-id="7064e-248">Part 2: Update web app options</span></span>

<span data-ttu-id="7064e-249">O [Serviço de Aplicações do Azure](/azure/app-service/overview) oferece um serviço de alojamento na Web altamente dimensionável e com correção automática.</span><span class="sxs-lookup"><span data-stu-id="7064e-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Serviço de Aplicações do Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="7064e-251">Mapeie um nome DNS personalizado existente para Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="7064e-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="7064e-252">Utilize um **registo CNAME** e um **registo A** para mapear um nome DNS personalizado para o Serviço de Aplicações.</span><span class="sxs-lookup"><span data-stu-id="7064e-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="7064e-253">Mapear um nome DNS personalizado já existente para as Aplicações Web do Azure</span><span class="sxs-lookup"><span data-stu-id="7064e-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="7064e-254">Utilize um CNAME para todos os nomes DNS personalizados, exceto um domínio raiz (por exemplo, northwind.com).</span><span class="sxs-lookup"><span data-stu-id="7064e-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="7064e-255">Para migrar um site em direto e o respetivo nome de domínio DNS para o Serviço de Aplicações, veja [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain) (Migrar um nome DNS ativo para o Serviço de Aplicações do Azure).</span><span class="sxs-lookup"><span data-stu-id="7064e-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="7064e-256">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="7064e-256">Prerequisites</span></span>

<span data-ttu-id="7064e-257">Para completar esta solução:</span><span class="sxs-lookup"><span data-stu-id="7064e-257">To complete this solution:</span></span>

- <span data-ttu-id="7064e-258">[Crie uma aplicação de Serviço de Aplicações,](/azure/app-service/)ou use uma aplicação criada para outra solução.</span><span class="sxs-lookup"><span data-stu-id="7064e-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="7064e-259">Compre um nome de domínio e garanta o acesso ao registo DNS para o fornecedor de domínio.</span><span class="sxs-lookup"><span data-stu-id="7064e-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="7064e-260">Atualize o ficheiro da zona DNS para o domínio.</span><span class="sxs-lookup"><span data-stu-id="7064e-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="7064e-261">A Azure AD verificará a propriedade do nome de domínio personalizado.</span><span class="sxs-lookup"><span data-stu-id="7064e-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="7064e-262">Utilize [o Azure DNS](/azure/dns/dns-getstarted-portal) para registos DNS Azure/Office 365/external DNS dentro do Azure, ou adicione a entrada de DNS [num registo DNS diferente](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="7064e-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

- <span data-ttu-id="7064e-263">Registe um domínio personalizado com um registo público.</span><span class="sxs-lookup"><span data-stu-id="7064e-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="7064e-264">Inicie sessão na entidade de registo de nome de domínio para o domínio.</span><span class="sxs-lookup"><span data-stu-id="7064e-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="7064e-265">(Um administrador aprovado pode ser necessário para fazer atualizações de DNS.)</span><span class="sxs-lookup"><span data-stu-id="7064e-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="7064e-266">Atualize o ficheiro da zona DNS para o domínio adicionando a entrada DNS fornecida pelo Azure AD.</span><span class="sxs-lookup"><span data-stu-id="7064e-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="7064e-267">Por exemplo, para adicionar entradas de DNS para northwindcloud.com e www \. northwindcloud.com, configurar as definições de DNS para o domínio raiz northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="7064e-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="7064e-268">Um nome de domínio pode ser adquirido usando o [portal Azure](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="7064e-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="7064e-269">Para mapear um nome DNS personalizado para uma aplicação Web, o [plano do Serviço de Aplicações](https://azure.microsoft.com/pricing/details/app-service/) dessa aplicação tem de ser um escalão pago (**Partilhado**, **Básico**, **Standard** ou **Premium**).</span><span class="sxs-lookup"><span data-stu-id="7064e-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="7064e-270">Criar e mapear registos CNAME e A</span><span class="sxs-lookup"><span data-stu-id="7064e-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="7064e-271">Aceder a registos DNS com o fornecedor de domínios</span><span class="sxs-lookup"><span data-stu-id="7064e-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="7064e-272">Utilize o Azure DNS para configurar um nome DNS personalizado para aplicações Web Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="7064e-273">Para obter mais informações, veja [Utilizar o DNS do Azure para oferecer definições de domínio personalizado para um serviço do Azure](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="7064e-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="7064e-274">Inscreva-se no site do fornecedor principal.</span><span class="sxs-lookup"><span data-stu-id="7064e-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="7064e-275">Localize a página para gerir os registos DNS.</span><span class="sxs-lookup"><span data-stu-id="7064e-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="7064e-276">Cada fornecedor de domínio tem a sua própria interface de registos DNS.</span><span class="sxs-lookup"><span data-stu-id="7064e-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="7064e-277">Procure áreas do site com os nomes **Nome de Domínio**, **DNS** ou **Gestão de Servidor de Nomes**.</span><span class="sxs-lookup"><span data-stu-id="7064e-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="7064e-278">A página de registos DNS pode ser visualizada nos **meus domínios**.</span><span class="sxs-lookup"><span data-stu-id="7064e-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="7064e-279">Encontre o **link**denominado ficheiro Zone , **DNS Records**ou **configuração Avançada**.</span><span class="sxs-lookup"><span data-stu-id="7064e-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="7064e-280">A captura de ecrã seguinte mostra um exemplo de uma página de registos DNS:</span><span class="sxs-lookup"><span data-stu-id="7064e-280">The following screenshot is an example of a DNS records page:</span></span>

![Página de registos DNS de exemplo](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="7064e-282">No Registo de Nomes de Domínio, **selecione Adicionar ou Criar** para criar um registo.</span><span class="sxs-lookup"><span data-stu-id="7064e-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="7064e-283">Alguns fornecedores têm ligações diferentes para adicionar diferentes tipos de registos.</span><span class="sxs-lookup"><span data-stu-id="7064e-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="7064e-284">Consulte a documentação do fornecedor.</span><span class="sxs-lookup"><span data-stu-id="7064e-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="7064e-285">Adicione um registo CNAME para mapear um subdomínio ao nome de anfitrião padrão da aplicação.</span><span class="sxs-lookup"><span data-stu-id="7064e-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="7064e-286">Para o \. exemplo de domínio www northwindcloud.com, adicione um registo CNAME que mapeie o nome para `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="7064e-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="7064e-287">Depois de adicionar o CNAME, a página de registos DNS parece ser o seguinte exemplo:</span><span class="sxs-lookup"><span data-stu-id="7064e-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Navegação do portal para a aplicação do Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="7064e-289">Ativar o mapeamento de registos CNAME no Azure</span><span class="sxs-lookup"><span data-stu-id="7064e-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="7064e-290">Num novo separador, inscreva-se no portal Azure.</span><span class="sxs-lookup"><span data-stu-id="7064e-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="7064e-291">Aceda a Serviços Aplicacionais.</span><span class="sxs-lookup"><span data-stu-id="7064e-291">Go to App Services.</span></span>

3. <span data-ttu-id="7064e-292">Selecione aplicativo web.</span><span class="sxs-lookup"><span data-stu-id="7064e-292">Select web app.</span></span>

4. <span data-ttu-id="7064e-293">No painel de navegação esquerdo da página da aplicação no portal do Azure, selecione **Domínios personalizados**.</span><span class="sxs-lookup"><span data-stu-id="7064e-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="7064e-294">Selecione o **+** ícone ao lado do nome de **anfitrião.**</span><span class="sxs-lookup"><span data-stu-id="7064e-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="7064e-295">Digite o nome de domínio totalmente qualificado, como `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="7064e-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="7064e-296">Selecione **Validar**.</span><span class="sxs-lookup"><span data-stu-id="7064e-296">Select **Validate**.</span></span>

8. <span data-ttu-id="7064e-297">Se indicado, adicione registos adicionais de outros tipos `A` (ou `TXT` ) aos registos DNS dos registos de nome de domínio.</span><span class="sxs-lookup"><span data-stu-id="7064e-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="7064e-298">A Azure fornecerá os valores e tipos destes registos:</span><span class="sxs-lookup"><span data-stu-id="7064e-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="7064e-299">a.</span><span class="sxs-lookup"><span data-stu-id="7064e-299">a.</span></span>  <span data-ttu-id="7064e-300">Um registo **A**, para mapear o endereço IP da aplicação.</span><span class="sxs-lookup"><span data-stu-id="7064e-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="7064e-301">b.</span><span class="sxs-lookup"><span data-stu-id="7064e-301">b.</span></span>  <span data-ttu-id="7064e-302">Um registo **TXT**, para mapear para o nome de anfitrião predefinido da aplicação, `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="7064e-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="7064e-303">O Serviço de Aplicações utiliza este registo apenas no momento da configuração para verificar a propriedade personalizada do domínio.</span><span class="sxs-lookup"><span data-stu-id="7064e-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="7064e-304">Após verificação, elimine o registo TXT.</span><span class="sxs-lookup"><span data-stu-id="7064e-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="7064e-305">Complete esta tarefa no separador de registo de domínio e revalida até que o botão **de nome de anfitrião Add** seja ativado.</span><span class="sxs-lookup"><span data-stu-id="7064e-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="7064e-306">Certifique-se de que **o tipo de registo hostname** está definido para **CNAME** (www.example.com ou qualquer subdomínio).</span><span class="sxs-lookup"><span data-stu-id="7064e-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="7064e-307">Selecione **Adicionar nome de anfitrião**.</span><span class="sxs-lookup"><span data-stu-id="7064e-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="7064e-308">Digite o nome de domínio totalmente qualificado, como `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="7064e-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="7064e-309">Selecione **Validar**.</span><span class="sxs-lookup"><span data-stu-id="7064e-309">Select **Validate**.</span></span> <span data-ttu-id="7064e-310">O **Add** é ativado.</span><span class="sxs-lookup"><span data-stu-id="7064e-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="7064e-311">Certifique-se de que **o tipo de gravação do hostname** está definido para um **recorde** (example.com).</span><span class="sxs-lookup"><span data-stu-id="7064e-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="7064e-312">**Adicione o nome de anfitrião**.</span><span class="sxs-lookup"><span data-stu-id="7064e-312">**Add hostname**.</span></span>

    <span data-ttu-id="7064e-313">Pode levar algum tempo para que os novos anfitriões sejam refletidos na página de **domínios personalizados** da aplicação.</span><span class="sxs-lookup"><span data-stu-id="7064e-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="7064e-314">Experimente atualizar o browser para atualizar os dados.</span><span class="sxs-lookup"><span data-stu-id="7064e-314">Try refreshing the browser to update the data.</span></span>
  
    ![Domínios personalizados](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="7064e-316">Se houver um erro, aparecerá uma notificação de erro de verificação na parte inferior da página.</span><span class="sxs-lookup"><span data-stu-id="7064e-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Erro de verificação de domínio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="7064e-318">Os passos acima podem ser repetidos para mapear um domínio wildcard \* (.northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="7064e-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="7064e-319">Isto permite a adição de quaisquer subdomínios adicionais a este serviço de aplicações sem ter de criar um registo CNAME separado para cada um.</span><span class="sxs-lookup"><span data-stu-id="7064e-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="7064e-320">Siga as instruções do registo para configurar esta definição.</span><span class="sxs-lookup"><span data-stu-id="7064e-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="7064e-321">Teste num browser</span><span class="sxs-lookup"><span data-stu-id="7064e-321">Test in a browser</span></span>

<span data-ttu-id="7064e-322">Navegue pelo(s) nome(s) DNS configurado anteriormente (por exemplo, `northwindcloud.com` ou `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="7064e-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="7064e-323">Parte 3: Bind a custom SSL cert</span><span class="sxs-lookup"><span data-stu-id="7064e-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="7064e-324">Nesta parte, iremos:</span><span class="sxs-lookup"><span data-stu-id="7064e-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="7064e-325">Ligue o certificado SSL personalizado ao Serviço de Aplicações.</span><span class="sxs-lookup"><span data-stu-id="7064e-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="7064e-326">Imponha HTTPS para a aplicação.</span><span class="sxs-lookup"><span data-stu-id="7064e-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="7064e-327">Automatizar a ligação do certificado SSL com scripts.</span><span class="sxs-lookup"><span data-stu-id="7064e-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="7064e-328">Se necessário, obtenha um certificado SSL de cliente no portal Azure e ligue-o à aplicação web.</span><span class="sxs-lookup"><span data-stu-id="7064e-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="7064e-329">Para mais informações, consulte o [tutorial dos Certificados de Serviço de Aplicações.](/azure/app-service/web-sites-purchase-ssl-web-site)</span><span class="sxs-lookup"><span data-stu-id="7064e-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="7064e-330">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="7064e-330">Prerequisites</span></span>

<span data-ttu-id="7064e-331">Para completar esta solução:</span><span class="sxs-lookup"><span data-stu-id="7064e-331">To complete this  solution:</span></span>

- [<span data-ttu-id="7064e-332">Crie uma aplicação de Serviço de Aplicações.</span><span class="sxs-lookup"><span data-stu-id="7064e-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="7064e-333">Mapeie um nome DNS personalizado para a sua aplicação web.</span><span class="sxs-lookup"><span data-stu-id="7064e-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="7064e-334">Adquira um certificado SSL a uma autoridade de certificados fidedignos e utilize a chave para assinar o pedido.</span><span class="sxs-lookup"><span data-stu-id="7064e-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="7064e-335">Requisitos do certificado SSL</span><span class="sxs-lookup"><span data-stu-id="7064e-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="7064e-336">Para utilizar um certificado no Serviço de Aplicações, aquele tem de cumprir os seguintes requisitos:</span><span class="sxs-lookup"><span data-stu-id="7064e-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="7064e-337">Assinado por uma autoridade de certificado de confiança.</span><span class="sxs-lookup"><span data-stu-id="7064e-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="7064e-338">Exportado como um ficheiro PFX protegido por palavra-passe.</span><span class="sxs-lookup"><span data-stu-id="7064e-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="7064e-339">Contém chave privada com pelo menos 2048 bits de comprimento.</span><span class="sxs-lookup"><span data-stu-id="7064e-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="7064e-340">Contém todos os certificados intermédios na cadeia de certificados.</span><span class="sxs-lookup"><span data-stu-id="7064e-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="7064e-341">**Os certificados de Criptografia da Curva Elíptica (ECC)** funcionam com o Serviço de Aplicações, mas não estão incluídos neste guia.</span><span class="sxs-lookup"><span data-stu-id="7064e-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="7064e-342">Consulte uma autoridade de certificação para assistência na criação de certificados ECC.</span><span class="sxs-lookup"><span data-stu-id="7064e-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="7064e-343">Preparar a aplicação web</span><span class="sxs-lookup"><span data-stu-id="7064e-343">Prepare the web app</span></span>

<span data-ttu-id="7064e-344">Para vincular um certificado SSL personalizado à aplicação web, o [plano de Serviço de Aplicações](https://azure.microsoft.com/pricing/details/app-service/) deve estar no nível **Básico,** **Standard**ou **Premium.**</span><span class="sxs-lookup"><span data-stu-id="7064e-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="7064e-345">Iniciar sessão no Azure</span><span class="sxs-lookup"><span data-stu-id="7064e-345">Sign in to Azure</span></span>

1. <span data-ttu-id="7064e-346">Abra o [portal Azure](https://portal.azure.com/) e vá para a aplicação web.</span><span class="sxs-lookup"><span data-stu-id="7064e-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="7064e-347">A partir do menu esquerdo, selecione **Serviços de Aplicações**e, em seguida, selecione o nome da aplicação web.</span><span class="sxs-lookup"><span data-stu-id="7064e-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Selecione aplicativo web no portal Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="7064e-349">Verificar o escalão de preço</span><span class="sxs-lookup"><span data-stu-id="7064e-349">Check the pricing tier</span></span>

1. <span data-ttu-id="7064e-350">Na navegação à esquerda da página da aplicação web, percorra a secção **Definições** e selecione **Scale up (Plano de Serviço de Aplicações)**.</span><span class="sxs-lookup"><span data-stu-id="7064e-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menu de escala na aplicação web](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="7064e-352">Certifique-se de que a aplicação web não está no nível **Gratuito** ou **Partilhado.**</span><span class="sxs-lookup"><span data-stu-id="7064e-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="7064e-353">O nível atual da aplicação web é destacado numa caixa azul escura.</span><span class="sxs-lookup"><span data-stu-id="7064e-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Verifique o nível de preços na aplicação web](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="7064e-355">O SSL personalizado não é suportado no nível **Gratuito** ou **Partilhado.**</span><span class="sxs-lookup"><span data-stu-id="7064e-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="7064e-356">Para aumentar a escala, siga os passos na secção seguinte ou na página de **nível de preços** e salte para [o Upload e prenda o certificado SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="7064e-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="7064e-357">Aumentar verticalmente o seu plano do Serviço de Aplicações</span><span class="sxs-lookup"><span data-stu-id="7064e-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="7064e-358">Selecione um dos escalões **Básico**, **Standard** ou **Premium**.</span><span class="sxs-lookup"><span data-stu-id="7064e-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="7064e-359">Selecione **Selecionar**.</span><span class="sxs-lookup"><span data-stu-id="7064e-359">Select **Select**.</span></span>

![Escolha o nível de preços para a sua aplicação web](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="7064e-361">A operação de escala fica completa quando a notificação é apresentada.</span><span class="sxs-lookup"><span data-stu-id="7064e-361">The scale operation is complete when notification is displayed.</span></span>

![Notificação para “aumentar verticalmente”](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="7064e-363">Ligue o seu certificado SSL e funda certificados intermédios</span><span class="sxs-lookup"><span data-stu-id="7064e-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="7064e-364">Fundir vários certificados na cadeia.</span><span class="sxs-lookup"><span data-stu-id="7064e-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="7064e-365">**Abra cada certificado** que recebeu num editor de texto.</span><span class="sxs-lookup"><span data-stu-id="7064e-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="7064e-366">Criar um ficheiro para o certificado fundido chamado *mergedcertate.crt*.</span><span class="sxs-lookup"><span data-stu-id="7064e-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="7064e-367">Num editor de texto, copie o conteúdo de cada certificado para este ficheiro.</span><span class="sxs-lookup"><span data-stu-id="7064e-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="7064e-368">A ordem dos seus certificados deve seguir a ordem na cadeia de certificados, a começar no seu certificado e a terminar no certificado de raiz.</span><span class="sxs-lookup"><span data-stu-id="7064e-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="7064e-369">O aspeto é igual ao do exemplo abaixo:</span><span class="sxs-lookup"><span data-stu-id="7064e-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="7064e-370">Exportar o certificado para PFX</span><span class="sxs-lookup"><span data-stu-id="7064e-370">Export certificate to PFX</span></span>

<span data-ttu-id="7064e-371">Exportar o certificado SSL fundido com a chave privada gerada pelo certificado.</span><span class="sxs-lookup"><span data-stu-id="7064e-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="7064e-372">Um ficheiro chave privado é criado através do OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="7064e-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="7064e-373">Para exportar o certificado para PFX, executar o seguinte comando e substituir os espaços reservados `<private-key-file>` e pelo caminho chave privado e o ficheiro de certificado `<merged-certificate-file>` fundido:</span><span class="sxs-lookup"><span data-stu-id="7064e-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="7064e-374">Quando solicitado, defina uma senha de exportação para o upload do seu certificado SSL para o Serviço de Aplicações mais tarde.</span><span class="sxs-lookup"><span data-stu-id="7064e-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="7064e-375">Quando o IIS ou **Certreq.exe** forem utilizados para gerar o pedido de certificado, instale o certificado numa máquina local e, em seguida, [exporte o certificado para PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="7064e-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="7064e-376">Faça o upload do certificado SSL</span><span class="sxs-lookup"><span data-stu-id="7064e-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="7064e-377">Selecione **as definições SSL** na navegação à esquerda da aplicação web.</span><span class="sxs-lookup"><span data-stu-id="7064e-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="7064e-378">Selecione **o Certificado de Upload**.</span><span class="sxs-lookup"><span data-stu-id="7064e-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="7064e-379">No **Ficheiro de Certificado PFX,** selecione o ficheiro PFX.</span><span class="sxs-lookup"><span data-stu-id="7064e-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="7064e-380">Na **palavra-passe do Certificado,** digite a palavra-passe criada ao exportar o ficheiro PFX.</span><span class="sxs-lookup"><span data-stu-id="7064e-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="7064e-381">Selecione **Carregar**.</span><span class="sxs-lookup"><span data-stu-id="7064e-381">Select **Upload**.</span></span>

    ![Certificado SSL de upload](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="7064e-383">Quando o Serviço de Aplicações termina o upload do certificado, aparece na página de **definições SSL.**</span><span class="sxs-lookup"><span data-stu-id="7064e-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Definições de SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="7064e-385">Vincular o seu certificado SSL</span><span class="sxs-lookup"><span data-stu-id="7064e-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="7064e-386">Na secção **de encadernações SSL,** selecione **Adicionar encadernação**.</span><span class="sxs-lookup"><span data-stu-id="7064e-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="7064e-387">Se o certificado tiver sido carregado, mas não aparecer em nomes de domínio no dropdown **do Nome Anfitrião,** tente refrescar a página do navegador.</span><span class="sxs-lookup"><span data-stu-id="7064e-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="7064e-388">Na página **Add SSL Binding,** utilize as descidas para selecionar o nome de domínio para garantir e o certificado a utilizar.</span><span class="sxs-lookup"><span data-stu-id="7064e-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="7064e-389">Em **Tipo de SSL**, selecione se vai utilizar [**SSL baseado em Indicação do Nome de Servidor (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) ou baseado em IP.</span><span class="sxs-lookup"><span data-stu-id="7064e-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="7064e-390">**SSL baseado em SNI:** Podem ser adicionadas ligações SSL baseadas em SNI múltiplas.</span><span class="sxs-lookup"><span data-stu-id="7064e-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="7064e-391">Esta opção permite utilizar vários certificados SSL para proteger múltiplos domínios no mesmo endereço IP.</span><span class="sxs-lookup"><span data-stu-id="7064e-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="7064e-392">Os browsers mais modernos (incluindo o Internet Explorer, o Chrome, o Firefox e o Opera) suportam SNI (encontre informações mais abrangentes sobre o suporte de browsers em [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication) [Indicação do Nome de Servidor]).</span><span class="sxs-lookup"><span data-stu-id="7064e-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="7064e-393">**SSL baseado em IP**: Apenas uma ligação SSL baseada em IP pode ser adicionada.</span><span class="sxs-lookup"><span data-stu-id="7064e-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="7064e-394">Esta opção permite utilizar apenas um certificado SSL para proteger um endereço IP público dedicado.</span><span class="sxs-lookup"><span data-stu-id="7064e-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="7064e-395">Para proteger vários domínios, fixe-os todos utilizando o mesmo certificado SSL.</span><span class="sxs-lookup"><span data-stu-id="7064e-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="7064e-396">O SSL baseado em IP é a opção tradicional para a ligação SSL.</span><span class="sxs-lookup"><span data-stu-id="7064e-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="7064e-397">Selecione **Adicionar Ligação**.</span><span class="sxs-lookup"><span data-stu-id="7064e-397">Select **Add Binding**.</span></span>

    ![Adicionar ligação SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="7064e-399">Quando o Serviço de Aplicações termina o upload do certificado, aparece nas secções **de encadernações SSL.**</span><span class="sxs-lookup"><span data-stu-id="7064e-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![As ligações SSL terminaram o upload](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="7064e-401">Remapia o recorde A para IP SSL</span><span class="sxs-lookup"><span data-stu-id="7064e-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="7064e-402">Se o SSL baseado em IP não for utilizado na aplicação web, salte para [testar HTTPS para o seu domínio personalizado](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="7064e-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="7064e-403">Por padrão, a aplicação web utiliza um endereço IP público partilhado.</span><span class="sxs-lookup"><span data-stu-id="7064e-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="7064e-404">Quando o certificado está ligado ao SSL baseado em IP, o App Service cria um novo e dedicado endereço IP para a aplicação web.</span><span class="sxs-lookup"><span data-stu-id="7064e-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="7064e-405">Quando um registo A é mapeado para a aplicação web, o registo de domínio deve ser atualizado com o endereço IP dedicado.</span><span class="sxs-lookup"><span data-stu-id="7064e-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="7064e-406">A página **de domínio personalizado** é atualizada com o novo endereço IP dedicado.</span><span class="sxs-lookup"><span data-stu-id="7064e-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="7064e-407">Copie este [endereço IP](/azure/app-service/app-service-web-tutorial-custom-domain)e, em seguida, remapia o [registo A](/azure/app-service/app-service-web-tutorial-custom-domain) para este novo endereço IP.</span><span class="sxs-lookup"><span data-stu-id="7064e-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="7064e-408">Tester HTTPS</span><span class="sxs-lookup"><span data-stu-id="7064e-408">Test HTTPS</span></span>

<span data-ttu-id="7064e-409">Em diferentes navegadores, vá `https://<your.custom.domain>` para garantir que a aplicação web é servida.</span><span class="sxs-lookup"><span data-stu-id="7064e-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![navegar para aplicativo web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="7064e-411">Se ocorrerem erros de validação de certificados, um certificado auto-assinado pode ser a causa, ou os certificados intermédios podem ter sido deixados de fora ao exportar para o ficheiro PFX.</span><span class="sxs-lookup"><span data-stu-id="7064e-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="7064e-412">Impor HTTPS</span><span class="sxs-lookup"><span data-stu-id="7064e-412">Enforce HTTPS</span></span>

<span data-ttu-id="7064e-413">Por predefinição, qualquer pessoa pode aceder à aplicação web utilizando HTTP.</span><span class="sxs-lookup"><span data-stu-id="7064e-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="7064e-414">Todos os pedidos HTTPS para a porta HTTPS podem ser redirecionados.</span><span class="sxs-lookup"><span data-stu-id="7064e-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="7064e-415">Na página da aplicação web, selecione **as definições SL**.</span><span class="sxs-lookup"><span data-stu-id="7064e-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="7064e-416">Em seguida, em **HTTPS Apenas**, selecione **Ativado**.</span><span class="sxs-lookup"><span data-stu-id="7064e-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Impor HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="7064e-418">Quando a operação estiver concluída, aceda a qualquer um dos URLs HTTP que apontam para a aplicação.</span><span class="sxs-lookup"><span data-stu-id="7064e-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="7064e-419">Por exemplo:</span><span class="sxs-lookup"><span data-stu-id="7064e-419">For example:</span></span>

- <span data-ttu-id="7064e-420">https://<app_name>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="7064e-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="7064e-421">Impor TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="7064e-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="7064e-422">A aplicação permite [tLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 por padrão, que já não é considerado seguro pelos padrões da indústria (como [PCI DSS).](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)</span><span class="sxs-lookup"><span data-stu-id="7064e-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="7064e-423">Para impor versões do TLS superiores, siga estes passos:</span><span class="sxs-lookup"><span data-stu-id="7064e-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="7064e-424">Na página da aplicação web, na navegação à esquerda, selecione **as definições SSL**.</span><span class="sxs-lookup"><span data-stu-id="7064e-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="7064e-425">Na **versão TLS,** selecione a versão TLS mínima.</span><span class="sxs-lookup"><span data-stu-id="7064e-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Impor TLS 1.1 ou 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="7064e-427">Criar um perfil do Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="7064e-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="7064e-428">Selecione Criar um perfil de gestor de tráfego de rede de **recursos**  >  **Networking**  >  **Traffic Manager profile**  >  **Criar**.</span><span class="sxs-lookup"><span data-stu-id="7064e-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="7064e-429">No painel **Criar perfil do Gestor de Tráfego**, preencha o seguinte:</span><span class="sxs-lookup"><span data-stu-id="7064e-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="7064e-430">Em **Nome,** forneça um nome para o perfil.</span><span class="sxs-lookup"><span data-stu-id="7064e-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="7064e-431">Este nome tem de ser único dentro da zona de manager.net de tráfego e resulta no nome DNS, trafficmanager.net, que é usado para aceder ao perfil de Gestor de Tráfego.</span><span class="sxs-lookup"><span data-stu-id="7064e-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="7064e-432">No **método de encaminhamento,** selecione o **método de encaminhamento Geográfico**.</span><span class="sxs-lookup"><span data-stu-id="7064e-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="7064e-433">Na **Subscrição**, selecione a subscrição sob a qual criar este perfil.</span><span class="sxs-lookup"><span data-stu-id="7064e-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="7064e-434">Em **Grupo de Recursos**, crie um grupo de recursos novo onde colocar este perfil.</span><span class="sxs-lookup"><span data-stu-id="7064e-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="7064e-435">Em **Localização do grupo de recursos**, selecione a localização do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="7064e-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="7064e-436">Esta definição refere-se à localização do grupo de recursos e não tem qualquer impacto no perfil do Gestor de Tráfego implantado globalmente.</span><span class="sxs-lookup"><span data-stu-id="7064e-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="7064e-437">Selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="7064e-437">Select **Create**.</span></span>

    7. <span data-ttu-id="7064e-438">Quando a implantação global do perfil de Gestor de Tráfego está completa, está listado no respetivo grupo de recursos como um dos recursos.</span><span class="sxs-lookup"><span data-stu-id="7064e-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Grupos de recursos na criação do perfil de Gestor de Tráfego](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="7064e-440">Adicionar pontos finais do Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="7064e-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="7064e-441">Na barra de pesquisa do portal, procure o nome de **perfil do Gestor de Tráfego** criado na secção anterior e selecione o perfil do gestor de tráfego nos resultados apresentados.</span><span class="sxs-lookup"><span data-stu-id="7064e-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="7064e-442">No **perfil de Gestor de Tráfego,** na secção **Definições,** selecione **Pontos de final**.</span><span class="sxs-lookup"><span data-stu-id="7064e-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="7064e-443">Selecione **Adicionar**.</span><span class="sxs-lookup"><span data-stu-id="7064e-443">Select **Add**.</span></span>

4. <span data-ttu-id="7064e-444">Adicionando o Azure Stack Hub Endpoint.</span><span class="sxs-lookup"><span data-stu-id="7064e-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="7064e-445">Para **o tipo**, selecione ponto final **externo**.</span><span class="sxs-lookup"><span data-stu-id="7064e-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="7064e-446">Forneça um **Nome** para este ponto final, idealmente o nome do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="7064e-447">Para um nome de domínio totalmente qualificado **(FQDN),** utilize o URL externo para a App Web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7064e-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="7064e-448">Em Geo-mapeamento, selecione uma região/continente onde o recurso está localizado.</span><span class="sxs-lookup"><span data-stu-id="7064e-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="7064e-449">Por exemplo, **a Europa.**</span><span class="sxs-lookup"><span data-stu-id="7064e-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="7064e-450">Sob a queda país/região que aparece, selecione o país que se aplica a este ponto final.</span><span class="sxs-lookup"><span data-stu-id="7064e-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="7064e-451">Por exemplo, **a Alemanha.**</span><span class="sxs-lookup"><span data-stu-id="7064e-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="7064e-452">Mantenha a caixa **Adicionar como desativado** desmarcada.</span><span class="sxs-lookup"><span data-stu-id="7064e-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="7064e-453">Selecione **OK**.</span><span class="sxs-lookup"><span data-stu-id="7064e-453">Select **OK**.</span></span>

12. <span data-ttu-id="7064e-454">Adicionar o Azure Endpoint:</span><span class="sxs-lookup"><span data-stu-id="7064e-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="7064e-455">Para **tipo**, selecione **Azure endpoint**.</span><span class="sxs-lookup"><span data-stu-id="7064e-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="7064e-456">Forneça um **nome** para o ponto final.</span><span class="sxs-lookup"><span data-stu-id="7064e-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="7064e-457">Para **o tipo de recurso Alvo**, selecione App **Service**.</span><span class="sxs-lookup"><span data-stu-id="7064e-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="7064e-458">Para **o recurso Target**, **selecione Escolha um serviço de aplicações** para mostrar a listagem das Aplicações Web na mesma subscrição.</span><span class="sxs-lookup"><span data-stu-id="7064e-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="7064e-459">Em **Recurso,** escolha o serviço app utilizado como primeiro ponto final.</span><span class="sxs-lookup"><span data-stu-id="7064e-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="7064e-460">Em Geo-mapeamento, selecione uma região/continente onde o recurso está localizado.</span><span class="sxs-lookup"><span data-stu-id="7064e-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="7064e-461">Por exemplo, **América do Norte/América Central/Caraíbas.**</span><span class="sxs-lookup"><span data-stu-id="7064e-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="7064e-462">Sob a queda país/região que aparece, deixe este lugar em branco para selecionar todos os agrupamentos regionais acima.</span><span class="sxs-lookup"><span data-stu-id="7064e-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="7064e-463">Mantenha a caixa **Adicionar como desativado** desmarcada.</span><span class="sxs-lookup"><span data-stu-id="7064e-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="7064e-464">Selecione **OK**.</span><span class="sxs-lookup"><span data-stu-id="7064e-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="7064e-465">Crie pelo menos um ponto final com um âmbito geográfico de All (World) para servir como o ponto final padrão para o recurso.</span><span class="sxs-lookup"><span data-stu-id="7064e-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="7064e-466">Quando a adição de ambos os pontos finais está completa, são exibidas no **perfil de Gestor de Tráfego,** juntamente com o seu estado de monitorização como **Online**.</span><span class="sxs-lookup"><span data-stu-id="7064e-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Estado do ponto final do gestor de tráfego](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="7064e-468">Global Enterprise conta com capacidades de geo-distribuição da Azure</span><span class="sxs-lookup"><span data-stu-id="7064e-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="7064e-469">Direcionar o tráfego de dados através do Azure Traffic Manager e os pontos finais específicos da geografia permite que as empresas globais cumpram as normas regionais e mantenham os dados conformes e seguros, o que é crucial para o sucesso de locais de negócios locais e remotos.</span><span class="sxs-lookup"><span data-stu-id="7064e-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="7064e-470">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="7064e-470">Next steps</span></span>

- <span data-ttu-id="7064e-471">Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)</span><span class="sxs-lookup"><span data-stu-id="7064e-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
