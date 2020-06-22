---
title: Configurar identidade em nuvem híbrida para aplicativos Azure e Azure Stack Hub
description: Saiba como configurar a identidade híbrida em nuvem para aplicações Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911318"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="04b89-103">Configurar identidade em nuvem híbrida para aplicativos Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="04b89-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="04b89-104">Saiba como configurar uma identidade híbrida em nuvem para as suas aplicações Azure e Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="04b89-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="04b89-105">Tem duas opções para garantir o acesso às suas apps tanto no Azure como no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="04b89-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="04b89-106">Quando o Azure Stack Hub tem uma ligação contínua à internet, pode utilizar o Azure Ative Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="04b89-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="04b89-107">Quando o Azure Stack Hub é desligado da internet, pode utilizar os Serviços Federados do Diretório Azure (AD FS).</span><span class="sxs-lookup"><span data-stu-id="04b89-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="04b89-108">Utiliza os principais serviços para conceder acesso às suas aplicações Azure Stack Hub para implementação ou configuração utilizando o Azure Resource Manager no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="04b89-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="04b89-109">Nesta solução, você construirá um ambiente de amostra para:</span><span class="sxs-lookup"><span data-stu-id="04b89-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="04b89-110">Estabeleça uma identidade híbrida no Global Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="04b89-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="04b89-111">Recupere um símbolo para aceder à Azure Stack Hub API.</span><span class="sxs-lookup"><span data-stu-id="04b89-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="04b89-112">Tem de ter permissões do operador Azure Stack Hub para os passos nesta solução.</span><span class="sxs-lookup"><span data-stu-id="04b89-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="04b89-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="04b89-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="04b89-114">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="04b89-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="04b89-115">O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que permite construir e implementar aplicações híbridas em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="04b89-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="04b89-116">O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas.</span><span class="sxs-lookup"><span data-stu-id="04b89-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="04b89-117">As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="04b89-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="04b89-118">Criar um diretor de serviço para a Azure AD no portal</span><span class="sxs-lookup"><span data-stu-id="04b89-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="04b89-119">Se implementou o Azure Stack Hub usando o Azure AD como loja de identidade, pode criar diretores de serviços tal como faz para o Azure.</span><span class="sxs-lookup"><span data-stu-id="04b89-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="04b89-120">[A utilização de uma identidade de aplicação para aceder](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) a recursos mostra-lhe como executar os passos através do portal.</span><span class="sxs-lookup"><span data-stu-id="04b89-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="04b89-121">Certifique-se de que tem as [permissões AD AZure necessárias](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="04b89-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="04b89-122">Criar um principal de serviço para AD FS usando PowerShell</span><span class="sxs-lookup"><span data-stu-id="04b89-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="04b89-123">Se implementou o Azure Stack Hub com AD FS, pode utilizar o PowerShell para criar um principal de serviço, atribuir uma função de acesso e iniciar sessão no PowerShell usando essa identidade.</span><span class="sxs-lookup"><span data-stu-id="04b89-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="04b89-124">[A utilização de uma identidade de aplicação para aceder a recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) mostra-lhe como executar os passos necessários usando o PowerShell.</span><span class="sxs-lookup"><span data-stu-id="04b89-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="04b89-125">Usando o Azure Stack Hub API</span><span class="sxs-lookup"><span data-stu-id="04b89-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="04b89-126">A solução [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md) acompanha-o através do processo de recuperação de um token para aceder à Azure Stack Hub API.</span><span class="sxs-lookup"><span data-stu-id="04b89-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="04b89-127">Ligue ao Azure Stack Hub usando PowerShell</span><span class="sxs-lookup"><span data-stu-id="04b89-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="04b89-128">O quickstart [para se levantar e funcionar com o PowerShell no Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) acompanha-o através dos passos necessários para instalar o Azure PowerShell e ligar-se à instalação do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="04b89-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="04b89-129">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="04b89-129">Prerequisites</span></span>

<span data-ttu-id="04b89-130">Precisa de uma instalação Azure Stack Hub ligada ao Azure AD com uma subscrição a que pode aceder.</span><span class="sxs-lookup"><span data-stu-id="04b89-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="04b89-131">Se não tiver uma instalação do Azure Stack Hub, pode utilizar estas instruções para configurar um [Kit de Desenvolvimento de Pilhas Azure Stack (ASDK)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="04b89-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="04b89-132">Ligue ao Azure Stack Hub usando código</span><span class="sxs-lookup"><span data-stu-id="04b89-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="04b89-133">Para ligar ao Azure Stack Hub utilizando código, utilize o API de pontos finais do Azure Resource Manager para obter os pontos finais de autenticação e gráfico para a instalação do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="04b89-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="04b89-134">Em seguida, autentica usando pedidos DE REST.</span><span class="sxs-lookup"><span data-stu-id="04b89-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="04b89-135">Pode encontrar uma aplicação de cliente de amostra no [GitHub.](https://github.com/shriramnat/HybridARMApplication)</span><span class="sxs-lookup"><span data-stu-id="04b89-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="04b89-136">A menos que o Azure SDK para o seu idioma de eleição suporte perfis API Azure, o SDK pode não funcionar com O Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="04b89-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="04b89-137">Para saber mais sobre perfis API Azure, consulte o artigo [de perfil de versão API de gestão.](/azure-stack/user/azure-stack-version-profiles.md)</span><span class="sxs-lookup"><span data-stu-id="04b89-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="04b89-138">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="04b89-138">Next steps</span></span>

- <span data-ttu-id="04b89-139">Para saber mais sobre como a identidade é tratada no Azure Stack Hub, consulte [a arquitetura de identidade para Azure Stack Hub.](/azure-stack/operator/azure-stack-identity-architecture.md)</span><span class="sxs-lookup"><span data-stu-id="04b89-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="04b89-140">Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](https://docs.microsoft.com/azure/architecture/patterns)</span><span class="sxs-lookup"><span data-stu-id="04b89-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
