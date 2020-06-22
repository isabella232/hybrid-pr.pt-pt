---
title: Padrão de retransmissão híbrido em Azure e Azure Stack Hub
description: Utilize o padrão de retransmissão híbrido em Azure e Azure Stack Hub para ligar aos recursos de borda protegidos por firewalls.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911150"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="c6b07-103">Padrão de retransmissão híbrido</span><span class="sxs-lookup"><span data-stu-id="c6b07-103">Hybrid relay pattern</span></span>

<span data-ttu-id="c6b07-104">Aprenda a ligar-se a recursos ou dispositivos de borda protegidos por firewalls utilizando o padrão de retransmissão híbrido e o Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="c6b07-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c6b07-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="c6b07-105">Context and problem</span></span>

<span data-ttu-id="c6b07-106">Os dispositivos edge estão frequentemente por trás de uma firewall corporativa ou dispositivo NAT.</span><span class="sxs-lookup"><span data-stu-id="c6b07-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="c6b07-107">Apesar de estarem seguros, podem não conseguir comunicar com a nuvem pública ou dispositivos de borda noutras redes corporativas.</span><span class="sxs-lookup"><span data-stu-id="c6b07-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="c6b07-108">Pode ser necessário expor certas portas e funcionalidades aos utilizadores na nuvem pública de forma segura.</span><span class="sxs-lookup"><span data-stu-id="c6b07-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="c6b07-109">Solução</span><span class="sxs-lookup"><span data-stu-id="c6b07-109">Solution</span></span>

<span data-ttu-id="c6b07-110">O padrão de retransmissão híbrido usa o Azure Relay para estabelecer um túnel WebSockets entre dois pontos finais que não conseguem comunicar diretamente.</span><span class="sxs-lookup"><span data-stu-id="c6b07-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="c6b07-111">Dispositivos que não estão no local, mas que precisam de se ligar a um ponto final no local, ligar-se-ão a um ponto final na nuvem pública.</span><span class="sxs-lookup"><span data-stu-id="c6b07-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="c6b07-112">Este ponto final irá redirecionar o tráfego em rotas predefinidas sobre um canal seguro.</span><span class="sxs-lookup"><span data-stu-id="c6b07-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="c6b07-113">Um ponto final dentro do ambiente no local recebe o tráfego e encaminha-o para o destino correto.</span><span class="sxs-lookup"><span data-stu-id="c6b07-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![arquitetura de solução de padrão de relé híbrido](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="c6b07-115">Eis como funciona o padrão de retransmissão híbrido:</span><span class="sxs-lookup"><span data-stu-id="c6b07-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="c6b07-116">Um dispositivo liga-se à máquina virtual (VM) em Azure, numa porta predefinida.</span><span class="sxs-lookup"><span data-stu-id="c6b07-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="c6b07-117">O trânsito é encaminhado para a Retransmissão Azure em Azure.</span><span class="sxs-lookup"><span data-stu-id="c6b07-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="c6b07-118">O VM on Azure Stack Hub, que já estabeleceu uma ligação de longa duração ao Azure Relay, recebe o tráfego e encaminha-o para o destino.</span><span class="sxs-lookup"><span data-stu-id="c6b07-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="c6b07-119">O serviço no local ou ponto final processa o pedido.</span><span class="sxs-lookup"><span data-stu-id="c6b07-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="c6b07-120">Componentes</span><span class="sxs-lookup"><span data-stu-id="c6b07-120">Components</span></span>

<span data-ttu-id="c6b07-121">Esta solução utiliza os seguintes componentes:</span><span class="sxs-lookup"><span data-stu-id="c6b07-121">This solution uses the following components:</span></span>

| <span data-ttu-id="c6b07-122">Camada</span><span class="sxs-lookup"><span data-stu-id="c6b07-122">Layer</span></span> | <span data-ttu-id="c6b07-123">Componente</span><span class="sxs-lookup"><span data-stu-id="c6b07-123">Component</span></span> | <span data-ttu-id="c6b07-124">Description</span><span class="sxs-lookup"><span data-stu-id="c6b07-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="c6b07-125">Azure</span><span class="sxs-lookup"><span data-stu-id="c6b07-125">Azure</span></span> | <span data-ttu-id="c6b07-126">VM do Azure</span><span class="sxs-lookup"><span data-stu-id="c6b07-126">Azure VM</span></span> | <span data-ttu-id="c6b07-127">Um Azure VM fornece um ponto final acessível ao público para o recurso no local.</span><span class="sxs-lookup"><span data-stu-id="c6b07-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="c6b07-128">Reencaminhamento do Azure</span><span class="sxs-lookup"><span data-stu-id="c6b07-128">Azure Relay</span></span> | <span data-ttu-id="c6b07-129">[Um Relé Azure](/azure/azure-relay/) fornece a infraestrutura para manter o túnel e a ligação entre o Azure VM e o Azure Stack Hub VM.</span><span class="sxs-lookup"><span data-stu-id="c6b07-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="c6b07-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="c6b07-130">Azure Stack Hub</span></span> | <span data-ttu-id="c6b07-131">Computação</span><span class="sxs-lookup"><span data-stu-id="c6b07-131">Compute</span></span> | <span data-ttu-id="c6b07-132">Um Azure Stack Hub VM fornece o lado do servidor do túnel de relé híbrido.</span><span class="sxs-lookup"><span data-stu-id="c6b07-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="c6b07-133">Armazenamento</span><span class="sxs-lookup"><span data-stu-id="c6b07-133">Storage</span></span> | <span data-ttu-id="c6b07-134">O cluster de motores AKS implantado no Azure Stack Hub fornece um motor escalável e resistente para executar o recipiente Face API.</span><span class="sxs-lookup"><span data-stu-id="c6b07-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="c6b07-135">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="c6b07-135">Issues and considerations</span></span>

<span data-ttu-id="c6b07-136">Considere os seguintes pontos ao decidir como implementar esta solução:</span><span class="sxs-lookup"><span data-stu-id="c6b07-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="c6b07-137">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="c6b07-137">Scalability</span></span>

<span data-ttu-id="c6b07-138">Este padrão só permite mapeamentos de porta de 1:1 no cliente e servidor.</span><span class="sxs-lookup"><span data-stu-id="c6b07-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="c6b07-139">Por exemplo, se a porta 80 estiver em túnel para um serviço no ponto final do Azure, não pode ser utilizada para outro serviço.</span><span class="sxs-lookup"><span data-stu-id="c6b07-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="c6b07-140">Os mapeamentos portuários devem ser planeados em conformidade.</span><span class="sxs-lookup"><span data-stu-id="c6b07-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="c6b07-141">O Relé Azure e os VMs devem ser devidamente dimensionados para lidar com o tráfego.</span><span class="sxs-lookup"><span data-stu-id="c6b07-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="c6b07-142">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="c6b07-142">Availability</span></span>

<span data-ttu-id="c6b07-143">Estes túneis e ligações não são redundantes.</span><span class="sxs-lookup"><span data-stu-id="c6b07-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="c6b07-144">Para garantir uma elevada disponibilidade, pode querer implementar código de verificação de erros.</span><span class="sxs-lookup"><span data-stu-id="c6b07-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="c6b07-145">Outra opção é ter um pool de VMs ligados a Azure Relé atrás de um equilibrador de carga.</span><span class="sxs-lookup"><span data-stu-id="c6b07-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="c6b07-146">Capacidade de gestão</span><span class="sxs-lookup"><span data-stu-id="c6b07-146">Manageability</span></span>

<span data-ttu-id="c6b07-147">Esta solução pode abranger muitos dispositivos e locais, o que pode ficar instável.</span><span class="sxs-lookup"><span data-stu-id="c6b07-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="c6b07-148">Os serviços IoT da Azure podem automaticamente trazer novos locais e dispositivos on-line e mantê-los atualizados.</span><span class="sxs-lookup"><span data-stu-id="c6b07-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="c6b07-149">Segurança</span><span class="sxs-lookup"><span data-stu-id="c6b07-149">Security</span></span>

<span data-ttu-id="c6b07-150">Este padrão, tal como mostrado, permite o acesso sem restrições a uma porta num dispositivo interno a partir da borda.</span><span class="sxs-lookup"><span data-stu-id="c6b07-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="c6b07-151">Considere adicionar um mecanismo de autenticação ao serviço no dispositivo interno ou em frente ao ponto final do relé híbrido.</span><span class="sxs-lookup"><span data-stu-id="c6b07-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c6b07-152">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="c6b07-152">Next steps</span></span>

<span data-ttu-id="c6b07-153">Para saber mais sobre os tópicos introduzidos neste artigo:</span><span class="sxs-lookup"><span data-stu-id="c6b07-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="c6b07-154">Este padrão usa Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="c6b07-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="c6b07-155">Para mais informações, consulte a documentação do [Relé Azure.](/azure/azure-relay/)</span><span class="sxs-lookup"><span data-stu-id="c6b07-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="c6b07-156">Consulte [considerações de design de aplicações híbridas](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e obter respostas a quaisquer perguntas adicionais.</span><span class="sxs-lookup"><span data-stu-id="c6b07-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="c6b07-157">Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="c6b07-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="c6b07-158">Quando estiver pronto para testar o exemplo da solução, continue com o guia de implementação da [solução de retransmissão Híbrida](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="c6b07-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="c6b07-159">O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.</span><span class="sxs-lookup"><span data-stu-id="c6b07-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>