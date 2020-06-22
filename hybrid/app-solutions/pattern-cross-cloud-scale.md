---
title: Padrão de escala de nuvem cruzada no Azure Stack Hub
description: Aprenda a construir uma aplicação de nuvem cruzada escalável no Azure e no Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911136"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="eda63-103">Padrão de escala de nuvem cruzada</span><span class="sxs-lookup"><span data-stu-id="eda63-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="eda63-104">Adicione automaticamente recursos a uma aplicação existente para acomodar um aumento de carga.</span><span class="sxs-lookup"><span data-stu-id="eda63-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="eda63-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="eda63-105">Context and problem</span></span>

<span data-ttu-id="eda63-106">A sua aplicação não consegue aumentar a capacidade de atender a aumentos inesperados da procura.</span><span class="sxs-lookup"><span data-stu-id="eda63-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="eda63-107">Esta falta de escalabilidade resulta em utilizadores que não chegam à aplicação durante os tempos de utilização máximo.</span><span class="sxs-lookup"><span data-stu-id="eda63-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="eda63-108">A aplicação pode servir um número fixo de utilizadores.</span><span class="sxs-lookup"><span data-stu-id="eda63-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="eda63-109">As empresas globais requerem aplicações seguras, fiáveis e disponíveis baseadas na nuvem.</span><span class="sxs-lookup"><span data-stu-id="eda63-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="eda63-110">Responder ao aumento da procura e utilizar as infraestruturas adequadas para apoiar essa procura é fundamental.</span><span class="sxs-lookup"><span data-stu-id="eda63-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="eda63-111">As empresas lutam para equilibrar custos e manutenção com segurança de dados empresariais, armazenamento e disponibilidade em tempo real.</span><span class="sxs-lookup"><span data-stu-id="eda63-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="eda63-112">Pode não ser capaz de executar a sua aplicação na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="eda63-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="eda63-113">No entanto, pode não ser economicamente viável para a empresa manter a capacidade necessária no seu ambiente no local para lidar com picos na procura da app.</span><span class="sxs-lookup"><span data-stu-id="eda63-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="eda63-114">Com este padrão, pode utilizar a elasticidade da nuvem pública com a sua solução no local.</span><span class="sxs-lookup"><span data-stu-id="eda63-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="eda63-115">Solução</span><span class="sxs-lookup"><span data-stu-id="eda63-115">Solution</span></span>

<span data-ttu-id="eda63-116">O padrão de escala de nuvem cruzada estende uma aplicação localizada numa nuvem local com recursos de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="eda63-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="eda63-117">O padrão é desencadeado por um aumento ou diminuição da procura, e respectivamente adiciona ou remove recursos na nuvem.</span><span class="sxs-lookup"><span data-stu-id="eda63-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="eda63-118">Estes recursos proporcionam redundância, disponibilidade rápida e encaminhamento geocompatíveis.</span><span class="sxs-lookup"><span data-stu-id="eda63-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Padrão de escala de nuvem cruzada](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="eda63-120">Este padrão aplica-se apenas aos componentes apátridas da sua aplicação.</span><span class="sxs-lookup"><span data-stu-id="eda63-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="eda63-121">Componentes</span><span class="sxs-lookup"><span data-stu-id="eda63-121">Components</span></span>

<span data-ttu-id="eda63-122">O padrão de escala de nuvem cruzada consiste nos seguintes componentes.</span><span class="sxs-lookup"><span data-stu-id="eda63-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="eda63-123">Fora da nuvem</span><span class="sxs-lookup"><span data-stu-id="eda63-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="eda63-124">Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="eda63-124">Traffic Manager</span></span>

<span data-ttu-id="eda63-125">No diagrama, este situa-se fora do grupo público de nuvens, mas teria de ser capaz de coordenar o tráfego tanto no centro de dados local como na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="eda63-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="eda63-126">O equilibrador oferece uma elevada disponibilidade para aplicação, monitorizando pontos finais e fornecendo redistribuição de falha quando necessário.</span><span class="sxs-lookup"><span data-stu-id="eda63-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="eda63-127">Sistema de Nomes de Domínio (DNS)</span><span class="sxs-lookup"><span data-stu-id="eda63-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="eda63-128">O Sistema de Nome de Domínio, ou DNS, é responsável por traduzir (ou resolver) um nome de website ou serviço para o seu endereço IP.</span><span class="sxs-lookup"><span data-stu-id="eda63-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="eda63-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="eda63-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="eda63-130">Servidor de construção hospedado</span><span class="sxs-lookup"><span data-stu-id="eda63-130">Hosted build server</span></span>

<span data-ttu-id="eda63-131">Um ambiente para hospedar o seu oleoduto de construção.</span><span class="sxs-lookup"><span data-stu-id="eda63-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="eda63-132">Recursos de aplicações</span><span class="sxs-lookup"><span data-stu-id="eda63-132">App resources</span></span>

<span data-ttu-id="eda63-133">Os recursos da aplicação precisam de ser capazes de escalar e escalar, como conjuntos de escala de máquinas virtuais e contentores.</span><span class="sxs-lookup"><span data-stu-id="eda63-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="eda63-134">Nome de domínio personalizado</span><span class="sxs-lookup"><span data-stu-id="eda63-134">Custom domain name</span></span>

<span data-ttu-id="eda63-135">Utilize um nome de domínio personalizado para pedidos de encaminhamento glob.</span><span class="sxs-lookup"><span data-stu-id="eda63-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="eda63-136">Endereços IP públicos</span><span class="sxs-lookup"><span data-stu-id="eda63-136">Public IP addresses</span></span>

<span data-ttu-id="eda63-137">Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="eda63-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="eda63-138">Nuvem local</span><span class="sxs-lookup"><span data-stu-id="eda63-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="eda63-139">Servidor de construção hospedado</span><span class="sxs-lookup"><span data-stu-id="eda63-139">Hosted build server</span></span>

<span data-ttu-id="eda63-140">Um ambiente para hospedar o seu oleoduto de construção.</span><span class="sxs-lookup"><span data-stu-id="eda63-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="eda63-141">Recursos de aplicações</span><span class="sxs-lookup"><span data-stu-id="eda63-141">App resources</span></span>

<span data-ttu-id="eda63-142">Os recursos da aplicação precisam da capacidade de escalar e escalar, como conjuntos de escala de máquinas virtuais e contentores.</span><span class="sxs-lookup"><span data-stu-id="eda63-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="eda63-143">Nome de domínio personalizado</span><span class="sxs-lookup"><span data-stu-id="eda63-143">Custom domain name</span></span>

<span data-ttu-id="eda63-144">Utilize um nome de domínio personalizado para pedidos de encaminhamento glob.</span><span class="sxs-lookup"><span data-stu-id="eda63-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="eda63-145">Endereços IP públicos</span><span class="sxs-lookup"><span data-stu-id="eda63-145">Public IP addresses</span></span>

<span data-ttu-id="eda63-146">Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="eda63-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="eda63-147">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="eda63-147">Issues and considerations</span></span>

<span data-ttu-id="eda63-148">Na altura de decidir como implementar este padrão, considere os seguintes pontos:</span><span class="sxs-lookup"><span data-stu-id="eda63-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="eda63-149">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="eda63-149">Scalability</span></span>

<span data-ttu-id="eda63-150">O componente-chave da escala de nuvens cruzadas é a capacidade de realizar o dimensionamento a pedido.</span><span class="sxs-lookup"><span data-stu-id="eda63-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="eda63-151">A escala deve ocorrer entre infraestruturas de nuvem pública e local e prestar um serviço consistente e fiável de acordo com a procura.</span><span class="sxs-lookup"><span data-stu-id="eda63-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="eda63-152">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="eda63-152">Availability</span></span>

<span data-ttu-id="eda63-153">Certifique-se de que as aplicações implementadas localmente são configuradas para alta disponibilidade através da configuração de hardware no local e implementação de software.</span><span class="sxs-lookup"><span data-stu-id="eda63-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="eda63-154">Capacidade de gestão</span><span class="sxs-lookup"><span data-stu-id="eda63-154">Manageability</span></span>

<span data-ttu-id="eda63-155">O padrão de nuvem cruzada garante uma gestão perfeita e uma interface familiar entre ambientes.</span><span class="sxs-lookup"><span data-stu-id="eda63-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="eda63-156">Quando utilizar este padrão</span><span class="sxs-lookup"><span data-stu-id="eda63-156">When to use this pattern</span></span>

<span data-ttu-id="eda63-157">Utilize este padrão:</span><span class="sxs-lookup"><span data-stu-id="eda63-157">Use this pattern:</span></span>

- <span data-ttu-id="eda63-158">Quando precisa de aumentar a sua capacidade de aplicação com exigências inesperadas ou exigências periódicas na procura.</span><span class="sxs-lookup"><span data-stu-id="eda63-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="eda63-159">Quando não quer investir em recursos que só serão usados durante picos.</span><span class="sxs-lookup"><span data-stu-id="eda63-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="eda63-160">Pague pelo que usa.</span><span class="sxs-lookup"><span data-stu-id="eda63-160">Pay for what you use.</span></span>

<span data-ttu-id="eda63-161">Este padrão não é recomendado quando:</span><span class="sxs-lookup"><span data-stu-id="eda63-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="eda63-162">A sua solução requer que os utilizadores se conectem através da internet.</span><span class="sxs-lookup"><span data-stu-id="eda63-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="eda63-163">O seu negócio tem regulamentos locais que exigem que a ligação originária venha de uma chamada no local.</span><span class="sxs-lookup"><span data-stu-id="eda63-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="eda63-164">A sua rede experimenta estrangulamentos regulares que restringiriam o desempenho da escala.</span><span class="sxs-lookup"><span data-stu-id="eda63-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="eda63-165">O seu ambiente está desligado da internet e não consegue alcançar a nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="eda63-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="eda63-166">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="eda63-166">Next steps</span></span>

<span data-ttu-id="eda63-167">Para saber mais sobre os tópicos introduzidos neste artigo:</span><span class="sxs-lookup"><span data-stu-id="eda63-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="eda63-168">Consulte a visão geral do [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como funciona este equilibrador de carga de tráfego baseado em DNS.</span><span class="sxs-lookup"><span data-stu-id="eda63-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="eda63-169">Consulte [considerações de design de aplicações híbridas](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter respostas para quaisquer perguntas adicionais.</span><span class="sxs-lookup"><span data-stu-id="eda63-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="eda63-170">Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="eda63-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="eda63-171">Quando estiver pronto para testar o exemplo da solução, continue com o [guia de implementação da solução de escala cross-cloud](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="eda63-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="eda63-172">O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.</span><span class="sxs-lookup"><span data-stu-id="eda63-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="eda63-173">Aprende-se a criar uma solução cross-cloud para fornecer um processo manualmente desencadeado para mudar de uma aplicação web hospedada do Azure Stack Hub para uma aplicação web hospedada aZure.</span><span class="sxs-lookup"><span data-stu-id="eda63-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="eda63-174">Também aprende a utilizar autoscaling através do gestor de tráfego, garantindo uma utilidade de nuvem flexível e escalável quando está carregada.</span><span class="sxs-lookup"><span data-stu-id="eda63-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
