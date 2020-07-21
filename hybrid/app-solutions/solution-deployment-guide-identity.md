---
title: Configurar identidade em nuvem híbrida para aplicativos Azure e Azure Stack Hub
description: Saiba como configurar a identidade híbrida em nuvem para aplicações Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477257"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Configurar identidade em nuvem híbrida para aplicativos Azure e Azure Stack Hub

Saiba como configurar uma identidade híbrida em nuvem para as suas aplicações Azure e Azure Stack Hub.

Tem duas opções para garantir o acesso às suas apps tanto no Azure como no Azure Stack Hub.

 * Quando o Azure Stack Hub tem uma ligação contínua à internet, pode utilizar o Azure Ative Directory (Azure AD).
 * Quando o Azure Stack Hub é desligado da internet, pode utilizar os Serviços Federados do Diretório Azure (AD FS).

Utiliza os principais serviços para conceder acesso às suas aplicações Azure Stack Hub para implementação ou configuração utilizando o Azure Resource Manager no Azure Stack Hub.

Nesta solução, você construirá um ambiente de amostra para:

> [!div class="checklist"]
> - Estabeleça uma identidade híbrida no Global Azure e Azure Stack Hub
> - Recupere um símbolo para aceder à Azure Stack Hub API.

Tem de ter permissões do operador Azure Stack Hub para os passos nesta solução.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Criar um diretor de serviço para a Azure AD no portal

Se implementou o Azure Stack Hub usando o Azure AD como loja de identidade, pode criar diretores de serviços tal como faz para o Azure. [A utilização de uma identidade de aplicação para aceder](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) a recursos mostra-lhe como executar os passos através do portal. Certifique-se de que tem as [permissões AD AZure necessárias](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) antes de começar.

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Criar um principal de serviço para AD FS usando PowerShell

Se implementou o Azure Stack Hub com AD FS, pode utilizar o PowerShell para criar um principal de serviço, atribuir uma função de acesso e iniciar sessão no PowerShell usando essa identidade. [A utilização de uma identidade de aplicação para aceder a recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) mostra-lhe como executar os passos necessários usando o PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Usando o Azure Stack Hub API

A solução [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md) acompanha-o através do processo de recuperação de um token para aceder à Azure Stack Hub API.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Ligue ao Azure Stack Hub usando PowerShell

O quickstart [para se levantar e funcionar com o PowerShell no Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) acompanha-o através dos passos necessários para instalar o Azure PowerShell e ligar-se à instalação do Azure Stack Hub.

### <a name="prerequisites"></a>Pré-requisitos

Precisa de uma instalação Azure Stack Hub ligada ao Azure AD com uma subscrição a que pode aceder. Se não tiver uma instalação do Azure Stack Hub, pode utilizar estas instruções para configurar um [Kit de Desenvolvimento de Pilhas Azure Stack (ASDK)](/azure-stack/asdk/asdk-install.md).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Ligue ao Azure Stack Hub usando código

Para ligar ao Azure Stack Hub utilizando código, utilize o API de pontos finais do Azure Resource Manager para obter os pontos finais de autenticação e gráfico para a instalação do Azure Stack Hub. Em seguida, autentica usando pedidos DE REST. Pode encontrar uma aplicação de cliente de amostra no [GitHub.](https://github.com/shriramnat/HybridARMApplication)

>[!Note]
>A menos que o Azure SDK para o seu idioma de eleição suporte perfis API Azure, o SDK pode não funcionar com O Azure Stack Hub. Para saber mais sobre perfis API Azure, consulte o artigo [de perfil de versão API de gestão.](/azure-stack/user/azure-stack-version-profiles.md)

## <a name="next-steps"></a>Passos seguintes

- Para saber mais sobre como a identidade é tratada no Azure Stack Hub, consulte [a arquitetura de identidade para Azure Stack Hub.](/azure-stack/operator/azure-stack-identity-architecture.md)
- Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)
