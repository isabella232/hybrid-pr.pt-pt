---
title: Padrão de aplicativo geo-distribuído no Azure Stack Hub
description: Saiba mais sobre o padrão de aplicação geo-distribuído para a borda inteligente usando Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910884"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="4e934-103">Padrão de aplicativo geo-distribuído</span><span class="sxs-lookup"><span data-stu-id="4e934-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="4e934-104">Saiba como fornecer pontos finais de aplicativos em várias regiões e encaminhar o tráfego de utilizadores com base nas necessidades de localização e conformidade.</span><span class="sxs-lookup"><span data-stu-id="4e934-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="4e934-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="4e934-105">Context and problem</span></span>

<span data-ttu-id="4e934-106">Organizações com geografias de grande alcance esforçam-se por distribuir de forma segura e precisa e permitir o acesso aos dados, garantindo ao mesmo tempo os níveis de segurança, conformidade e desempenho necessários por utilizador, localização e dispositivo além-fronteiras.</span><span class="sxs-lookup"><span data-stu-id="4e934-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="4e934-107">Solução</span><span class="sxs-lookup"><span data-stu-id="4e934-107">Solution</span></span>

<span data-ttu-id="4e934-108">O padrão de encaminhamento de tráfego geográfico Azure Stack Hub, ou aplicações geo-distribuídas, permite que o tráfego seja direcionado para pontos finais específicos com base em várias métricas.</span><span class="sxs-lookup"><span data-stu-id="4e934-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="4e934-109">A criação de um Gestor de Tráfego com encaminhamento geográfico e configuração de pontos finais encaminha o tráfego para pontos finais com base nos requisitos regionais, regulação corporativa e internacional e necessidades de dados.</span><span class="sxs-lookup"><span data-stu-id="4e934-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Padrão geo-distribuído](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="4e934-111">Componentes</span><span class="sxs-lookup"><span data-stu-id="4e934-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="4e934-112">Fora da nuvem</span><span class="sxs-lookup"><span data-stu-id="4e934-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="4e934-113">Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="4e934-113">Traffic Manager</span></span>

<span data-ttu-id="4e934-114">No diagrama, o Traffic Manager está localizado fora da nuvem pública, mas precisa de ser capaz de coordenar o tráfego tanto no centro de dados local como na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="4e934-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="4e934-115">O equilibrador encaminha o tráfego para locais geográficos.</span><span class="sxs-lookup"><span data-stu-id="4e934-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="4e934-116">Sistema de Nomes de Domínio (DNS)</span><span class="sxs-lookup"><span data-stu-id="4e934-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="4e934-117">O Sistema de Nome de Domínio, ou DNS, é responsável por traduzir (ou resolver) um nome de website ou serviço para o seu endereço IP.</span><span class="sxs-lookup"><span data-stu-id="4e934-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="4e934-118">Cloud pública</span><span class="sxs-lookup"><span data-stu-id="4e934-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="4e934-119">Ponto final da nuvem</span><span class="sxs-lookup"><span data-stu-id="4e934-119">Cloud Endpoint</span></span>

<span data-ttu-id="4e934-120">Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="4e934-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="4e934-121">Nuvens locais</span><span class="sxs-lookup"><span data-stu-id="4e934-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="4e934-122">Ponto final local</span><span class="sxs-lookup"><span data-stu-id="4e934-122">Local endpoint</span></span>

<span data-ttu-id="4e934-123">Os endereços IP públicos são usados para encaminhar o tráfego de entrada através do gestor de tráfego para o ponto final de recursos de aplicações de nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="4e934-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="4e934-124">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="4e934-124">Issues and considerations</span></span>

<span data-ttu-id="4e934-125">Na altura de decidir como implementar este padrão, considere os seguintes pontos:</span><span class="sxs-lookup"><span data-stu-id="4e934-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="4e934-126">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="4e934-126">Scalability</span></span>

<span data-ttu-id="4e934-127">O padrão lida com o encaminhamento geográfico do tráfego em vez de escalar para fazer face aos aumentos de tráfego.</span><span class="sxs-lookup"><span data-stu-id="4e934-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="4e934-128">No entanto, pode combinar este padrão com outras soluções Azure e no local.</span><span class="sxs-lookup"><span data-stu-id="4e934-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="4e934-129">Por exemplo, este padrão pode ser usado com o Padrão de escala de nuvem cruzada.</span><span class="sxs-lookup"><span data-stu-id="4e934-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="4e934-130">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="4e934-130">Availability</span></span>

<span data-ttu-id="4e934-131">Certifique-se de que as aplicações implementadas localmente são configuradas para alta disponibilidade através da configuração de hardware no local e implementação de software.</span><span class="sxs-lookup"><span data-stu-id="4e934-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="4e934-132">Capacidade de gestão</span><span class="sxs-lookup"><span data-stu-id="4e934-132">Manageability</span></span>

<span data-ttu-id="4e934-133">O padrão garante uma gestão perfeita e uma interface familiar entre ambientes.</span><span class="sxs-lookup"><span data-stu-id="4e934-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="4e934-134">Quando utilizar este padrão</span><span class="sxs-lookup"><span data-stu-id="4e934-134">When to use this pattern</span></span>

- <span data-ttu-id="4e934-135">A minha organização tem agências internacionais que exigem políticas de segurança e distribuição regionais personalizadas.</span><span class="sxs-lookup"><span data-stu-id="4e934-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="4e934-136">Cada um dos escritórios da minha organização retira dados de empregados, negócios e instalações, exigindo atividade de reporte de acordo com os regulamentos locais e fuso horário.</span><span class="sxs-lookup"><span data-stu-id="4e934-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="4e934-137">Requisitos de alta escala podem ser cumpridos escalando horizontalmente aplicações, com várias implementações de aplicações sendo feitas dentro de uma única região e em todas as regiões para lidar com requisitos de carga extrema.</span><span class="sxs-lookup"><span data-stu-id="4e934-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="4e934-138">As aplicações devem estar altamente disponíveis e responder aos pedidos dos clientes mesmo em interrupções de uma região única.</span><span class="sxs-lookup"><span data-stu-id="4e934-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="4e934-139">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="4e934-139">Next steps</span></span>

<span data-ttu-id="4e934-140">Para saber mais sobre os tópicos introduzidos neste artigo:</span><span class="sxs-lookup"><span data-stu-id="4e934-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="4e934-141">Consulte a visão geral do [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) para saber mais sobre como funciona este equilibrador de carga de tráfego baseado em DNS.</span><span class="sxs-lookup"><span data-stu-id="4e934-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="4e934-142">Consulte [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter respostas para quaisquer perguntas adicionais.</span><span class="sxs-lookup"><span data-stu-id="4e934-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="4e934-143">Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="4e934-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="4e934-144">Quando estiver pronto para testar o exemplo da solução, continue com o [guia de implementação da solução de aplicações geo-distribuídas.](solution-deployment-guide-geo-distributed.md)</span><span class="sxs-lookup"><span data-stu-id="4e934-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="4e934-145">O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.</span><span class="sxs-lookup"><span data-stu-id="4e934-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="4e934-146">Aprende-se a direcionar o tráfego para pontos finais específicos, com base em várias métricas utilizando o padrão de aplicação geo-distribuído.</span><span class="sxs-lookup"><span data-stu-id="4e934-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="4e934-147">A criação de um perfil de Gestor de Tráfego com configuração geográfica de encaminhamento e ponto final garante que a informação é encaminhada para pontos finais com base nos requisitos regionais, na regulação corporativa e internacional e nas necessidades dos seus dados.</span><span class="sxs-lookup"><span data-stu-id="4e934-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
