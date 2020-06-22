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
# <a name="hybrid-relay-pattern"></a>Padrão de retransmissão híbrido

Aprenda a ligar-se a recursos ou dispositivos de borda protegidos por firewalls utilizando o padrão de retransmissão híbrido e o Azure Relay.

## <a name="context-and-problem"></a>Contexto e problema

Os dispositivos edge estão frequentemente por trás de uma firewall corporativa ou dispositivo NAT. Apesar de estarem seguros, podem não conseguir comunicar com a nuvem pública ou dispositivos de borda noutras redes corporativas. Pode ser necessário expor certas portas e funcionalidades aos utilizadores na nuvem pública de forma segura.

## <a name="solution"></a>Solução

O padrão de retransmissão híbrido usa o Azure Relay para estabelecer um túnel WebSockets entre dois pontos finais que não conseguem comunicar diretamente. Dispositivos que não estão no local, mas que precisam de se ligar a um ponto final no local, ligar-se-ão a um ponto final na nuvem pública. Este ponto final irá redirecionar o tráfego em rotas predefinidas sobre um canal seguro. Um ponto final dentro do ambiente no local recebe o tráfego e encaminha-o para o destino correto.

![arquitetura de solução de padrão de relé híbrido](media/pattern-hybrid-relay/solution-architecture.png)

Eis como funciona o padrão de retransmissão híbrido:

1. Um dispositivo liga-se à máquina virtual (VM) em Azure, numa porta predefinida.
2. O trânsito é encaminhado para a Retransmissão Azure em Azure.
3. O VM on Azure Stack Hub, que já estabeleceu uma ligação de longa duração ao Azure Relay, recebe o tráfego e encaminha-o para o destino.
4. O serviço no local ou ponto final processa o pedido.

## <a name="components"></a>Componentes

Esta solução utiliza os seguintes componentes:

| Camada | Componente | Description |
|----------|-----------|-------------|
| Azure | VM do Azure | Um Azure VM fornece um ponto final acessível ao público para o recurso no local. |
| | Reencaminhamento do Azure | [Um Relé Azure](/azure/azure-relay/) fornece a infraestrutura para manter o túnel e a ligação entre o Azure VM e o Azure Stack Hub VM.|
| Azure Stack Hub | Computação | Um Azure Stack Hub VM fornece o lado do servidor do túnel de relé híbrido. |
| | Armazenamento | O cluster de motores AKS implantado no Azure Stack Hub fornece um motor escalável e resistente para executar o recipiente Face API.|

## <a name="issues-and-considerations"></a>Problemas e considerações

Considere os seguintes pontos ao decidir como implementar esta solução:

### <a name="scalability"></a>Escalabilidade

Este padrão só permite mapeamentos de porta de 1:1 no cliente e servidor. Por exemplo, se a porta 80 estiver em túnel para um serviço no ponto final do Azure, não pode ser utilizada para outro serviço. Os mapeamentos portuários devem ser planeados em conformidade. O Relé Azure e os VMs devem ser devidamente dimensionados para lidar com o tráfego.

### <a name="availability"></a>Disponibilidade

Estes túneis e ligações não são redundantes. Para garantir uma elevada disponibilidade, pode querer implementar código de verificação de erros. Outra opção é ter um pool de VMs ligados a Azure Relé atrás de um equilibrador de carga.

### <a name="manageability"></a>Capacidade de gestão

Esta solução pode abranger muitos dispositivos e locais, o que pode ficar instável. Os serviços IoT da Azure podem automaticamente trazer novos locais e dispositivos on-line e mantê-los atualizados.

### <a name="security"></a>Segurança

Este padrão, tal como mostrado, permite o acesso sem restrições a uma porta num dispositivo interno a partir da borda. Considere adicionar um mecanismo de autenticação ao serviço no dispositivo interno ou em frente ao ponto final do relé híbrido.

## <a name="next-steps"></a>Passos seguintes

Para saber mais sobre os tópicos introduzidos neste artigo:

- Este padrão usa Azure Relay. Para mais informações, consulte a documentação do [Relé Azure.](/azure/azure-relay/)
- Consulte [considerações de design de aplicações híbridas](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e obter respostas a quaisquer perguntas adicionais.
- Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.

Quando estiver pronto para testar o exemplo da solução, continue com o guia de implementação da [solução de retransmissão Híbrida](https://aka.ms/hybridrelaydeployment). O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.