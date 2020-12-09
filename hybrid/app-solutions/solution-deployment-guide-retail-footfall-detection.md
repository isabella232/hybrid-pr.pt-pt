---
title: Implementar solução de deteção de rodapé baseada em IA no Azure e no Azure Stack Hub
description: Saiba como implementar uma solução de deteção de rodapé baseada em IA para analisar o tráfego de visitantes em lojas de retalho usando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901495"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="06669-103">Implementar uma solução de deteção de base de IA utilizando o Azure e o Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="06669-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="06669-104">Este artigo descreve como implementar uma solução baseada em IA que gere insights de ações do mundo real usando Azure Stack Hub e o Kit Dev Dev Visão Personalizada.</span><span class="sxs-lookup"><span data-stu-id="06669-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="06669-105">Nesta solução, aprende-se a:</span><span class="sxs-lookup"><span data-stu-id="06669-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="06669-106">Implementar pacotes de aplicações nativas em nuvem (CNAB) na borda.</span><span class="sxs-lookup"><span data-stu-id="06669-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="06669-107">Implemente uma aplicação que abra por entre limites de nuvens.</span><span class="sxs-lookup"><span data-stu-id="06669-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="06669-108">Utilize o Kit de Dev Vision AI Personalizado para inferência na borda.</span><span class="sxs-lookup"><span data-stu-id="06669-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="06669-109">![Diagrama de pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="06669-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="06669-110">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="06669-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="06669-111">O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="06669-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="06669-112">O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas.</span><span class="sxs-lookup"><span data-stu-id="06669-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="06669-113">As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="06669-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="06669-114">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="06669-114">Prerequisites</span></span>

<span data-ttu-id="06669-115">Antes de começar com este guia de implantação, certifique-se de que:</span><span class="sxs-lookup"><span data-stu-id="06669-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="06669-116">Reveja o tópico do [padrão de deteção de Footfall.](pattern-retail-footfall-detection.md)</span><span class="sxs-lookup"><span data-stu-id="06669-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="06669-117">Obtenha acesso do utilizador a um Kit de Desenvolvimento de Pilhas Azure (ASDK) ou a instância integrada do Azure Stack Hub, com:</span><span class="sxs-lookup"><span data-stu-id="06669-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="06669-118">O [Serviço de Aplicações Azure no fornecedor de recursos Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview.md) instalado.</span><span class="sxs-lookup"><span data-stu-id="06669-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="06669-119">Precisa de acesso do operador à sua instância do Azure Stack Hub ou trabalhar com o seu administrador para instalar.</span><span class="sxs-lookup"><span data-stu-id="06669-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="06669-120">Uma subscrição de uma oferta que fornece o Serviço de Aplicações e quota de armazenamento.</span><span class="sxs-lookup"><span data-stu-id="06669-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="06669-121">Precisa de acesso do operador para criar uma oferta.</span><span class="sxs-lookup"><span data-stu-id="06669-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="06669-122">Obtenha acesso a uma subscrição do Azure.</span><span class="sxs-lookup"><span data-stu-id="06669-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="06669-123">Se não tiver uma assinatura Azure, inscreva-se para uma [conta de teste gratuita](https://azure.microsoft.com/free/) antes de começar.</span><span class="sxs-lookup"><span data-stu-id="06669-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="06669-124">Crie dois diretores de serviço no seu diretório:</span><span class="sxs-lookup"><span data-stu-id="06669-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="06669-125">Um conjunto para uso com recursos Azure, com acesso no âmbito de subscrição Azure.</span><span class="sxs-lookup"><span data-stu-id="06669-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="06669-126">Um conjunto para uso com recursos do Azure Stack Hub, com acesso no âmbito de subscrição do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="06669-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="06669-127">Para saber mais sobre a criação de diretores de serviços e autorizar o acesso, consulte [utilizar uma identidade de aplicação para aceder aos recursos.](/azure-stack/operator/azure-stack-create-service-principals.md)</span><span class="sxs-lookup"><span data-stu-id="06669-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="06669-128">Se preferir utilizar o Azure CLI, consulte [criar um rente-chefe de serviço Azure com Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span><span class="sxs-lookup"><span data-stu-id="06669-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="06669-129">Implementar serviços cognitivos Azure em Azure ou Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="06669-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="06669-130">Primeiro, [saiba mais sobre os Serviços Cognitivos.](https://azure.microsoft.com/services/cognitive-services/)</span><span class="sxs-lookup"><span data-stu-id="06669-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="06669-131">Em [seguida, visite implementar os Serviços Cognitivos Azure para o Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) para implantar serviços cognitivos no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="06669-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="06669-132">Primeiro tem de se inscrever para ter acesso à pré-visualização.</span><span class="sxs-lookup"><span data-stu-id="06669-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="06669-133">Clone ou descarregue um Azure Custom Vision AI Dev Kit não configurado.</span><span class="sxs-lookup"><span data-stu-id="06669-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="06669-134">Para mais detalhes, consulte o [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="06669-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="06669-135">Inscreva-se numa conta Power BI.</span><span class="sxs-lookup"><span data-stu-id="06669-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="06669-136">Uma chave de subscrição de Azure Cognitive Services Face API e URL de ponto final.</span><span class="sxs-lookup"><span data-stu-id="06669-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="06669-137">Pode obter os dois com o teste gratuito [dos Serviços Cognitivos.](https://azure.microsoft.com/try/cognitive-services/?api=face-api)</span><span class="sxs-lookup"><span data-stu-id="06669-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="06669-138">Ou, siga as instruções na [conta Criar um Serviço Cognitivo.](/azure/cognitive-services/cognitive-services-apis-create-account)</span><span class="sxs-lookup"><span data-stu-id="06669-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="06669-139">Instalar os seguintes recursos de desenvolvimento:</span><span class="sxs-lookup"><span data-stu-id="06669-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="06669-140">CLI 2.0 do Azure</span><span class="sxs-lookup"><span data-stu-id="06669-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="06669-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="06669-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="06669-142">[Porter, o](https://porter.sh/)que está a fazer?</span><span class="sxs-lookup"><span data-stu-id="06669-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="06669-143">Você usa Porter para implementar aplicativos em nuvem usando manifestos de pacote CNAB que são fornecidos para si.</span><span class="sxs-lookup"><span data-stu-id="06669-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="06669-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="06669-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="06669-145">Azure IoT Tools para Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="06669-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="06669-146">Extensão python para código de estúdio visual</span><span class="sxs-lookup"><span data-stu-id="06669-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="06669-147">Python</span><span class="sxs-lookup"><span data-stu-id="06669-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="06669-148">Implementar a aplicação de nuvem híbrida</span><span class="sxs-lookup"><span data-stu-id="06669-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="06669-149">Primeiro utiliza o Porter CLI para gerar um conjunto de credenciais e depois implementa a aplicação cloud.</span><span class="sxs-lookup"><span data-stu-id="06669-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="06669-150">Clone ou descarregue o código de amostra de solução a partir de https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="06669-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="06669-151">Porter irá gerar um conjunto de credenciais que automatizarão a implementação da app.</span><span class="sxs-lookup"><span data-stu-id="06669-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="06669-152">Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:</span><span class="sxs-lookup"><span data-stu-id="06669-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="06669-153">Um diretor de serviço para aceder aos recursos da Azure, incluindo o iD principal de serviço, chave, e DNS inquilino.</span><span class="sxs-lookup"><span data-stu-id="06669-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="06669-154">O ID de subscrição para a sua subscrição Azure.</span><span class="sxs-lookup"><span data-stu-id="06669-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="06669-155">Um diretor de serviço para aceder aos recursos do Azure Stack Hub, incluindo o iD principal de serviço, chave e DNS inquilino.</span><span class="sxs-lookup"><span data-stu-id="06669-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="06669-156">O ID de subscrição para a sua assinatura Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="06669-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="06669-157">Os seus serviços cognitivos Azure enfrentam a chave API e URL de ponto final de recurso.</span><span class="sxs-lookup"><span data-stu-id="06669-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="06669-158">Executar o processo de geração de credenciais Porter e seguir as instruções:</span><span class="sxs-lookup"><span data-stu-id="06669-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="06669-159">Porter também requer um conjunto de parâmetros para correr.</span><span class="sxs-lookup"><span data-stu-id="06669-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="06669-160">Crie um ficheiro de texto de parâmetro e introduza os seguintes pares de nome/valor.</span><span class="sxs-lookup"><span data-stu-id="06669-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="06669-161">Pergunte ao administrador do Azure Stack Hub se precisa de assistência com algum dos valores necessários.</span><span class="sxs-lookup"><span data-stu-id="06669-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="06669-162">O `resource suffix` valor é usado para garantir que os recursos da sua implantação têm nomes únicos em todo o Azure.</span><span class="sxs-lookup"><span data-stu-id="06669-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="06669-163">Deve ser uma cadeia única de letras e números, não mais do que 8 caracteres.</span><span class="sxs-lookup"><span data-stu-id="06669-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="06669-164">Guarde o ficheiro de texto e tome nota do seu percurso.</span><span class="sxs-lookup"><span data-stu-id="06669-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="06669-165">Está agora pronto para implementar a aplicação de nuvem híbrida usando o Porter.</span><span class="sxs-lookup"><span data-stu-id="06669-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="06669-166">Executar o comando de instalação e assistir à medida que os recursos são implantados para O Azure e Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="06669-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="06669-167">Uma vez concluída a implementação, tome nota dos seguintes valores:</span><span class="sxs-lookup"><span data-stu-id="06669-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="06669-168">A ligação da câmara.</span><span class="sxs-lookup"><span data-stu-id="06669-168">The camera's connection string.</span></span>
    - <span data-ttu-id="06669-169">A cadeia de ligação da conta de armazenamento de imagem.</span><span class="sxs-lookup"><span data-stu-id="06669-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="06669-170">Os nomes do grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="06669-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="06669-171">Preparar o Vision AI DevKit personalizado</span><span class="sxs-lookup"><span data-stu-id="06669-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="06669-172">Em seguida, configurar o Kit Dev Dev Vision AI personalizado, como mostrado no [quickstart Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="06669-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="06669-173">Também configura e testa a sua câmara, utilizando a cadeia de ligação fornecida no passo anterior.</span><span class="sxs-lookup"><span data-stu-id="06669-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="06669-174">Implementar a aplicação da câmara</span><span class="sxs-lookup"><span data-stu-id="06669-174">Deploy the camera app</span></span>

<span data-ttu-id="06669-175">Utilize o Porter CLI para gerar um conjunto de credenciais e, em seguida, desloque a aplicação da câmara.</span><span class="sxs-lookup"><span data-stu-id="06669-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="06669-176">Porter irá gerar um conjunto de credenciais que automatizarão a implementação da app.</span><span class="sxs-lookup"><span data-stu-id="06669-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="06669-177">Antes de executar o comando de geração de credenciais, certifique-se de ter o seguinte disponível:</span><span class="sxs-lookup"><span data-stu-id="06669-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="06669-178">Um diretor de serviço para aceder aos recursos da Azure, incluindo o iD principal de serviço, chave, e DNS inquilino.</span><span class="sxs-lookup"><span data-stu-id="06669-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="06669-179">O ID de subscrição para a sua subscrição Azure.</span><span class="sxs-lookup"><span data-stu-id="06669-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="06669-180">A cadeia de ligação da conta de armazenamento de imagem fornecida quando implementou a aplicação cloud.</span><span class="sxs-lookup"><span data-stu-id="06669-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="06669-181">Executar o processo de geração de credenciais Porter e seguir as instruções:</span><span class="sxs-lookup"><span data-stu-id="06669-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="06669-182">Porter também requer um conjunto de parâmetros para correr.</span><span class="sxs-lookup"><span data-stu-id="06669-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="06669-183">Crie um ficheiro de texto de parâmetro e introduza o seguinte texto.</span><span class="sxs-lookup"><span data-stu-id="06669-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="06669-184">Pergunte ao administrador do Azure Stack Hub se não conhece alguns dos valores necessários.</span><span class="sxs-lookup"><span data-stu-id="06669-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="06669-185">O `deployment suffix` valor é usado para garantir que os recursos da sua implantação têm nomes únicos em todo o Azure.</span><span class="sxs-lookup"><span data-stu-id="06669-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="06669-186">Deve ser uma cadeia única de letras e números, não mais do que 8 caracteres.</span><span class="sxs-lookup"><span data-stu-id="06669-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="06669-187">Guarde o ficheiro de texto e tome nota do seu percurso.</span><span class="sxs-lookup"><span data-stu-id="06669-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="06669-188">Está agora pronto para implementar a aplicação da câmara usando o Porter.</span><span class="sxs-lookup"><span data-stu-id="06669-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="06669-189">Executar o comando de instalação e observar à medida que a implementação IoT Edge é criada.</span><span class="sxs-lookup"><span data-stu-id="06669-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="06669-190">Verifique se a implementação da câmara está completa visualizando o feed da câmara `https://<camera-ip>:3000/` em , onde está o endereço IP da `<camara-ip>` câmara.</span><span class="sxs-lookup"><span data-stu-id="06669-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="06669-191">Este passo pode demorar até 10 minutos.</span><span class="sxs-lookup"><span data-stu-id="06669-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="06669-192">Configurar Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="06669-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="06669-193">Agora que os dados estão a fluir para o Azure Stream Analytics a partir da câmara, precisamos de autorizar manualmente que comunique com o Power BI.</span><span class="sxs-lookup"><span data-stu-id="06669-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="06669-194">A partir do portal Azure, abra **todos os recursos,** e o *processo-footfall seu trabalho de \[ \] sufixo.*</span><span class="sxs-lookup"><span data-stu-id="06669-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="06669-195">Na secção **Topologia da Tarefa** do painel de tarefas do Stream Analytics, selecione a opção **Saídas**.</span><span class="sxs-lookup"><span data-stu-id="06669-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="06669-196">Selecione o **lavatório de saída de saída de tráfego.**</span><span class="sxs-lookup"><span data-stu-id="06669-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="06669-197">Selecione **Renovar a autorização** e iniciar sôm no seu Power BI account.</span><span class="sxs-lookup"><span data-stu-id="06669-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Renovar o pedido de autorização no Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="06669-199">Guarde as definições de saída.</span><span class="sxs-lookup"><span data-stu-id="06669-199">Save the output settings.</span></span>

6. <span data-ttu-id="06669-200">Vá ao **painel de visão geral** e selecione **Comece** a enviar dados para Power BI.</span><span class="sxs-lookup"><span data-stu-id="06669-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="06669-201">Selecione **Agora** para a hora de início da saída da tarefa e selecione **Iniciar**.</span><span class="sxs-lookup"><span data-stu-id="06669-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="06669-202">Pode ver o estado da tarefa na barra de notificação.</span><span class="sxs-lookup"><span data-stu-id="06669-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="06669-203">Criar um painel de instrumentos power BI</span><span class="sxs-lookup"><span data-stu-id="06669-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="06669-204">Assim que o trabalho tiver sucesso, vá ao [Power BI](https://powerbi.com/) e inscreva-se na sua conta de trabalho ou escola.</span><span class="sxs-lookup"><span data-stu-id="06669-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="06669-205">Se a consulta de trabalho stream Analytics estiver a desausar resultados, o conjunto *de dados de base* que criou existe no **separador Datasets.**</span><span class="sxs-lookup"><span data-stu-id="06669-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="06669-206">A partir do seu espaço de trabalho Power BI, selecione **+ Criar** para criar um novo painel chamado *Footfall Analysis.*</span><span class="sxs-lookup"><span data-stu-id="06669-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="06669-207">Na parte superior da janela, selecione **Adicionar mosaico**.</span><span class="sxs-lookup"><span data-stu-id="06669-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="06669-208">Em seguida, selecione **Dados de Transmissão em Fluxo Personalizados** e **Seguinte**.</span><span class="sxs-lookup"><span data-stu-id="06669-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="06669-209">Escolha o **conjunto de dados de rodapé** nos seus **conjuntos de dados**.</span><span class="sxs-lookup"><span data-stu-id="06669-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="06669-210">Selecione **o Cartão** a partir do **dropdown do tipo visualização** e adicione **a idade** aos **Campos**.</span><span class="sxs-lookup"><span data-stu-id="06669-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="06669-211">Selecione **Seguinte** para introduzir um nome para o mosaico e, em seguida, selecione **Aplicar** para criar o mosaico.</span><span class="sxs-lookup"><span data-stu-id="06669-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="06669-212">Pode adicionar campos e cartões adicionais conforme desejado.</span><span class="sxs-lookup"><span data-stu-id="06669-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="06669-213">Teste a sua solução</span><span class="sxs-lookup"><span data-stu-id="06669-213">Test Your Solution</span></span>

<span data-ttu-id="06669-214">Observe como os dados nos cartões que criou no Power BI mudam à medida que pessoas diferentes andam em frente à câmara.</span><span class="sxs-lookup"><span data-stu-id="06669-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="06669-215">As inferências podem demorar até 20 segundos a aparecer uma vez gravadas.</span><span class="sxs-lookup"><span data-stu-id="06669-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="06669-216">Remova a sua solução</span><span class="sxs-lookup"><span data-stu-id="06669-216">Remove Your Solution</span></span>

<span data-ttu-id="06669-217">Se quiser remover a sua solução, execute os seguintes comandos utilizando o Porter, utilizando os mesmos ficheiros de parâmetros que criou para a implementação:</span><span class="sxs-lookup"><span data-stu-id="06669-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="06669-218">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="06669-218">Next steps</span></span>

- <span data-ttu-id="06669-219">Saiba mais sobre [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="06669-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="06669-220">Reveja e proponha melhorias [ao código para esta amostra no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="06669-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
