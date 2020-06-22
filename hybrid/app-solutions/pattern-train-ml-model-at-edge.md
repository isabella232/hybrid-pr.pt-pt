---
title: Modelo de aprendizagem de máquina de trem no padrão de borda
description: Aprenda a fazer o treino de modelo de machine learning no limite com Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911097"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="23493-103">Modelo de aprendizagem de máquina de trem no padrão de borda</span><span class="sxs-lookup"><span data-stu-id="23493-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="23493-104">Gerem modelos portáteis de aprendizagem automática (ML) a partir de dados que só existem no local.</span><span class="sxs-lookup"><span data-stu-id="23493-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="23493-105">Contexto e problema</span><span class="sxs-lookup"><span data-stu-id="23493-105">Context and problem</span></span>

<span data-ttu-id="23493-106">Muitas organizações gostariam de desbloquear insights a partir dos seus dados no local ou do legado usando ferramentas que os seus cientistas de dados entendem.</span><span class="sxs-lookup"><span data-stu-id="23493-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="23493-107">[A Azure Machine Learning](/azure/machine-learning/) fornece ferramentas nativas em nuvem para treinar, sintonizar e implementar ml e modelos de aprendizagem profunda.</span><span class="sxs-lookup"><span data-stu-id="23493-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="23493-108">No entanto, alguns dados são muito grandes enviados para a nuvem ou não podem ser enviados para a nuvem por razões regulamentares.</span><span class="sxs-lookup"><span data-stu-id="23493-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="23493-109">Usando este padrão, os cientistas de dados podem usar a Azure Machine Learning para treinar modelos usando dados no local e computar.</span><span class="sxs-lookup"><span data-stu-id="23493-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="23493-110">Solução</span><span class="sxs-lookup"><span data-stu-id="23493-110">Solution</span></span>

<span data-ttu-id="23493-111">O treinamento no padrão de borda usa uma máquina virtual (VM) em execução no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="23493-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="23493-112">O VM está registado como um alvo de computação em Azure ML, permitindo-lhe aceder apenas aos dados disponíveis no local.</span><span class="sxs-lookup"><span data-stu-id="23493-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="23493-113">Neste caso, os dados são armazenados no armazenamento de bolhas do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="23493-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="23493-114">Uma vez treinado o modelo, é registado com Azure ML, contentorizado, e adicionado a um Registo de Contentores Azure para implantação.</span><span class="sxs-lookup"><span data-stu-id="23493-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="23493-115">Para esta iteração do padrão, o VM de treino Azure Stack Hub deve ser acessível através da internet pública.</span><span class="sxs-lookup"><span data-stu-id="23493-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="23493-116">[![Modelo de ML de trem na arquitetura de borda](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="23493-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="23493-117">Eis como funciona o padrão:</span><span class="sxs-lookup"><span data-stu-id="23493-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="23493-118">O Azure Stack Hub VM é implantado e registado como alvo de computação com a Azure ML.</span><span class="sxs-lookup"><span data-stu-id="23493-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="23493-119">Uma experiência é criada em Azure ML que usa o Azure Stack Hub VM como alvo de computação.</span><span class="sxs-lookup"><span data-stu-id="23493-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="23493-120">Uma vez treinado o modelo, é registado e contentorizado.</span><span class="sxs-lookup"><span data-stu-id="23493-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="23493-121">O modelo pode agora ser implantado em locais que estejam no local ou na nuvem.</span><span class="sxs-lookup"><span data-stu-id="23493-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="23493-122">Componentes</span><span class="sxs-lookup"><span data-stu-id="23493-122">Components</span></span>

<span data-ttu-id="23493-123">Esta solução utiliza os seguintes componentes:</span><span class="sxs-lookup"><span data-stu-id="23493-123">This solution uses the following components:</span></span>

| <span data-ttu-id="23493-124">Camada</span><span class="sxs-lookup"><span data-stu-id="23493-124">Layer</span></span> | <span data-ttu-id="23493-125">Componente</span><span class="sxs-lookup"><span data-stu-id="23493-125">Component</span></span> | <span data-ttu-id="23493-126">Description</span><span class="sxs-lookup"><span data-stu-id="23493-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="23493-127">Azure</span><span class="sxs-lookup"><span data-stu-id="23493-127">Azure</span></span> | <span data-ttu-id="23493-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="23493-128">Azure Machine Learning</span></span> | <span data-ttu-id="23493-129">[A Azure Machine Learning](/azure/machine-learning/) orquestra a formação do modelo ML.</span><span class="sxs-lookup"><span data-stu-id="23493-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="23493-130">Registo de Contentores do Azure</span><span class="sxs-lookup"><span data-stu-id="23493-130">Azure Container Registry</span></span> | <span data-ttu-id="23493-131">A Azure ML embala o modelo num recipiente e armazena-o num [registo de contentores Azure](/azure/container-registry/) para implantação.</span><span class="sxs-lookup"><span data-stu-id="23493-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="23493-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="23493-132">Azure Stack Hub</span></span> | <span data-ttu-id="23493-133">Serviço de Aplicações</span><span class="sxs-lookup"><span data-stu-id="23493-133">App Service</span></span> | <span data-ttu-id="23493-134">[O Azure Stack Hub com o App Service](/azure-stack/operator/azure-stack-app-service-overview) fornece a base para os componentes na borda.</span><span class="sxs-lookup"><span data-stu-id="23493-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="23493-135">Computação</span><span class="sxs-lookup"><span data-stu-id="23493-135">Compute</span></span> | <span data-ttu-id="23493-136">Um Azure Stack Hub VM que executa Ubuntu com Docker é usado para treinar o modelo ML.</span><span class="sxs-lookup"><span data-stu-id="23493-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="23493-137">Armazenamento</span><span class="sxs-lookup"><span data-stu-id="23493-137">Storage</span></span> | <span data-ttu-id="23493-138">Os dados privados podem ser hospedados no armazenamento de blob Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="23493-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="23493-139">Problemas e considerações</span><span class="sxs-lookup"><span data-stu-id="23493-139">Issues and considerations</span></span>

<span data-ttu-id="23493-140">Considere os seguintes pontos ao decidir como implementar esta solução:</span><span class="sxs-lookup"><span data-stu-id="23493-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="23493-141">Escalabilidade</span><span class="sxs-lookup"><span data-stu-id="23493-141">Scalability</span></span>

<span data-ttu-id="23493-142">Para permitir esta solução à escala, terá de criar um VM de tamanho adequado no Azure Stack Hub para treinar.</span><span class="sxs-lookup"><span data-stu-id="23493-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="23493-143">Disponibilidade</span><span class="sxs-lookup"><span data-stu-id="23493-143">Availability</span></span>

<span data-ttu-id="23493-144">Certifique-se de que os scripts de treino e o Azure Stack Hub VM têm acesso aos dados no local utilizados para o treino.</span><span class="sxs-lookup"><span data-stu-id="23493-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="23493-145">Capacidade de gestão</span><span class="sxs-lookup"><span data-stu-id="23493-145">Manageability</span></span>

<span data-ttu-id="23493-146">Certifique-se de que os modelos e experiências estão devidamente registados, versados e marcados para evitar confusões durante a implementação do modelo.</span><span class="sxs-lookup"><span data-stu-id="23493-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="23493-147">Segurança</span><span class="sxs-lookup"><span data-stu-id="23493-147">Security</span></span>

<span data-ttu-id="23493-148">Este padrão permite ao Azure ML aceder a possíveis dados sensíveis no local.</span><span class="sxs-lookup"><span data-stu-id="23493-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="23493-149">Certifique-se de que a conta utilizada para SSH no Azure Stack Hub VM tem uma palavra-passe forte e os scripts de treino não preservam ou carregam dados para a nuvem.</span><span class="sxs-lookup"><span data-stu-id="23493-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="23493-150">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="23493-150">Next steps</span></span>

<span data-ttu-id="23493-151">Para saber mais sobre os tópicos introduzidos neste artigo:</span><span class="sxs-lookup"><span data-stu-id="23493-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="23493-152">Consulte a [documentação de Aprendizagem automática Azure](/azure/machine-learning) para uma visão geral do ML e tópicos relacionados.</span><span class="sxs-lookup"><span data-stu-id="23493-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="23493-153">Consulte [o Registo de Contentores Azure](/azure/container-registry/) para aprender a construir, armazenar e gerir imagens para implantações de contentores.</span><span class="sxs-lookup"><span data-stu-id="23493-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="23493-154">Consulte o [Serviço de Aplicações no Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) para saber mais sobre o fornecedor de recursos e como implementar.</span><span class="sxs-lookup"><span data-stu-id="23493-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="23493-155">Consulte [considerações de design de aplicações híbridas](overview-app-design-considerations.md) para saber mais sobre as melhores práticas e para obter quaisquer perguntas adicionais respondidas.</span><span class="sxs-lookup"><span data-stu-id="23493-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="23493-156">Veja a [família de produtos e soluções Azure Stack](/azure-stack) para saber mais sobre todo o portfólio de produtos e soluções.</span><span class="sxs-lookup"><span data-stu-id="23493-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="23493-157">Quando estiver pronto para testar o exemplo da solução, continue com o [modelo ML do comboio no guia de implantação de bordas](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="23493-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="23493-158">O guia de implantação fornece instruções passo a passo para a implantação e teste dos seus componentes.</span><span class="sxs-lookup"><span data-stu-id="23493-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
