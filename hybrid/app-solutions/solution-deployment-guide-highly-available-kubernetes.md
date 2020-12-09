---
title: Implementar cluster Kubernetes altamente disponível no Azure Stack Hub
description: Saiba como implementar uma solução de cluster Kubernetes para uma alta disponibilidade utilizando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911738"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="25321-103">Implementar um cluster Kubernetes de alta disponibilidade no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="25321-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="25321-104">Este artigo irá mostrar-lhe como construir um ambiente de cluster Kubernetes altamente disponível, implantado em várias instâncias do Azure Stack Hub, em diferentes locais físicos.</span><span class="sxs-lookup"><span data-stu-id="25321-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="25321-105">Neste guia de implementação de soluções, aprende-se a:</span><span class="sxs-lookup"><span data-stu-id="25321-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="25321-106">Descarregue e prepare o motor AKS</span><span class="sxs-lookup"><span data-stu-id="25321-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="25321-107">Ligue-se ao VM do Ajudante do Motor AKS</span><span class="sxs-lookup"><span data-stu-id="25321-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="25321-108">Implementar um cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="25321-109">Ligue-se ao cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="25321-110">Ligue os gasodutos Azure ao cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="25321-111">Configurar a monitorização</span><span class="sxs-lookup"><span data-stu-id="25321-111">Configure monitoring</span></span>
> - <span data-ttu-id="25321-112">Implementar aplicação</span><span class="sxs-lookup"><span data-stu-id="25321-112">Deploy application</span></span>
> - <span data-ttu-id="25321-113">Aplicação de escala automática</span><span class="sxs-lookup"><span data-stu-id="25321-113">Autoscale application</span></span>
> - <span data-ttu-id="25321-114">Configurar o Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="25321-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="25321-115">Atualizar Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="25321-116">Kubernetes de escala</span><span class="sxs-lookup"><span data-stu-id="25321-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="25321-117">![Pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="25321-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="25321-118">Microsoft Azure Stack Hub é uma extensão do Azure.</span><span class="sxs-lookup"><span data-stu-id="25321-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="25321-119">O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.</span><span class="sxs-lookup"><span data-stu-id="25321-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="25321-120">O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas.</span><span class="sxs-lookup"><span data-stu-id="25321-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="25321-121">As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.</span><span class="sxs-lookup"><span data-stu-id="25321-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="25321-122">Pré-requisitos</span><span class="sxs-lookup"><span data-stu-id="25321-122">Prerequisites</span></span>

<span data-ttu-id="25321-123">Antes de começar com este guia de implantação, certifique-se de que:</span><span class="sxs-lookup"><span data-stu-id="25321-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="25321-124">Reveja o artigo [de padrão de cluster kubernetes de alta disponibilidade.](pattern-highly-available-kubernetes.md)</span><span class="sxs-lookup"><span data-stu-id="25321-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="25321-125">Reveja o conteúdo do [repositório GitHub,](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)que contém ativos adicionais referenciados neste artigo.</span><span class="sxs-lookup"><span data-stu-id="25321-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="25321-126">Ter uma conta que possa aceder ao portal de [utilizadores do Azure Stack Hub,](/azure-stack/user/azure-stack-use-portal)com pelo menos [permissões de "contribuinte".](/azure-stack/user/azure-stack-manage-permissions)</span><span class="sxs-lookup"><span data-stu-id="25321-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="25321-127">Baixar e preparar motor AKS</span><span class="sxs-lookup"><span data-stu-id="25321-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="25321-128">O Motor AKS é um binário que pode ser usado a partir de qualquer anfitrião Windows ou Linux que possa chegar aos pontos finais do Azure Stack Hub Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="25321-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="25321-129">Este guia descreve a implementação de um novo VM Linux (ou Windows) no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="25321-130">Será usado mais tarde quando o motor AKS implementar os clusters Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="25321-131">Também pode utilizar um Windows ou Linux VM existente para implantar um cluster Kubernetes no Azure Stack Hub utilizando o Motor AKS.</span><span class="sxs-lookup"><span data-stu-id="25321-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="25321-132">O processo passo a passo e os requisitos para o motor AKS estão documentados aqui:</span><span class="sxs-lookup"><span data-stu-id="25321-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="25321-133">[Instale o motor AKS no Linux no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (ou utilizando o [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="25321-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="25321-134">O Motor AKS é uma ferramenta auxiliar para implantar e operar clusters Kubernetes (em Azure e Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="25321-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="25321-135">Os detalhes e diferenças do motor AKS no Azure Stack Hub são descritos aqui:</span><span class="sxs-lookup"><span data-stu-id="25321-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="25321-136">O que é o motor AKS no Azure Stack Hub?</span><span class="sxs-lookup"><span data-stu-id="25321-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="25321-137">[Motor AKS no Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (no GitHub)</span><span class="sxs-lookup"><span data-stu-id="25321-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="25321-138">O ambiente da amostra utilizará o Terraform para automatizar a implantação do VM do motor AKS.</span><span class="sxs-lookup"><span data-stu-id="25321-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="25321-139">Você pode encontrar os [detalhes e código no companheiro GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="25321-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="25321-140">O resultado deste passo é um novo grupo de recursos no Azure Stack Hub que contém o VM do ajudante do motor AKS e recursos conexos:</span><span class="sxs-lookup"><span data-stu-id="25321-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Recursos VM do motor AKS no Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="25321-142">Se tiver de implantar o Motor AKS num ambiente desligado, [reveja as instâncias do Hub de Pilha de Azure para](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) saber mais.</span><span class="sxs-lookup"><span data-stu-id="25321-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="25321-143">No próximo passo, usaremos o recém-implantado VM do motor AKS para implantar um cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="25321-144">Ligue-se ao VM ajudante do motor AKS</span><span class="sxs-lookup"><span data-stu-id="25321-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="25321-145">Primeiro, deve ligar-se ao VM do ajudante do motor AKS anteriormente criado.</span><span class="sxs-lookup"><span data-stu-id="25321-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="25321-146">O VM deve ter um Endereço IP Público e deve estar acessível via SSH (Porta 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="25321-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Página geral do motor AKS VM](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="25321-148">Pode utilizar uma ferramenta à sua escolha como MobaXterm, puTTY ou PowerShell no Windows 10 para se ligar a um Linux VM utilizando SSH.</span><span class="sxs-lookup"><span data-stu-id="25321-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="25321-149">Depois de ligar, executar o comando `aks-engine` .</span><span class="sxs-lookup"><span data-stu-id="25321-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="25321-150">Vá às [versões suportadas do motor AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) para saber mais sobre as versões AKS Engine e Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![exemplo da linha de comando do motor aks](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="25321-152">Implementar um cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="25321-153">O próprio VM, ajudante do motor AKS, ainda não criou um cluster Kubernetes no nosso Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="25321-154">A criação do cluster é a primeira ação a tomar no VM do ajudante do motor AKS.</span><span class="sxs-lookup"><span data-stu-id="25321-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="25321-155">O processo passo a passo é documentado aqui:</span><span class="sxs-lookup"><span data-stu-id="25321-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="25321-156">Implementar um cluster Kubernetes com o motor AKS no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="25321-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="25321-157">O resultado final do `aks-engine deploy` comando e dos preparativos nos passos anteriores é um cluster Kubernetes totalmente apresentado implantado no espaço de inquilino da primeira instância do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="25321-158">O cluster em si é composto por componentes Azure IaaS como VMs, equilibradores de carga, VNets, discos, etc.</span><span class="sxs-lookup"><span data-stu-id="25321-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Cluster IaaS componentes Azure Stack Hub portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="25321-160">Equilibrador de carga Azure (K8s API Endpoint)</span><span class="sxs-lookup"><span data-stu-id="25321-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="25321-161">Nós operários (Piscina do Agente)</span><span class="sxs-lookup"><span data-stu-id="25321-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="25321-162">Mestre Nosdes</span><span class="sxs-lookup"><span data-stu-id="25321-162">Master Nodes</span></span>

<span data-ttu-id="25321-163">O aglomerado está a funcionar e no próximo passo vamos ligar-nos a ele.</span><span class="sxs-lookup"><span data-stu-id="25321-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="25321-164">Ligue-se ao cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="25321-165">Pode agora ligar-se ao cluster Kubernetes anteriormente criado, quer via SSH (utilizando a chave SSH especificada como parte da implantação) quer via `kubectl` (recomendada).</span><span class="sxs-lookup"><span data-stu-id="25321-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="25321-166">A ferramenta de linha de comando Kubernetes `kubectl` está disponível para Windows, Linux e macOS [aqui.](https://kubernetes.io/docs/tasks/tools/install-kubectl/)</span><span class="sxs-lookup"><span data-stu-id="25321-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="25321-167">Já está pré-instalado e configurado nos nós mestres do nosso aglomerado.</span><span class="sxs-lookup"><span data-stu-id="25321-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Execute kubectl no nó mestre](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="25321-169">Não é recomendado usar o nó mestre como uma caixa de salto para tarefas administrativas.</span><span class="sxs-lookup"><span data-stu-id="25321-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="25321-170">A `kubectl` configuração é armazenada `.kube/config` no(s) nó(s) principal, bem como no VM do motor AKS.</span><span class="sxs-lookup"><span data-stu-id="25321-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="25321-171">Pode copiar a configuração para uma máquina de administração com conectividade ao cluster Kubernetes e utilizar o `kubectl` comando lá.</span><span class="sxs-lookup"><span data-stu-id="25321-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="25321-172">O `.kube/config` ficheiro também é usado mais tarde para configurar uma ligação de serviço em Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="25321-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="25321-173">Mantenha estes ficheiros seguros porque contêm as credenciais para o seu cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="25321-174">Um intruso com acesso ao ficheiro tem informações suficientes para obter acesso ao administrador.</span><span class="sxs-lookup"><span data-stu-id="25321-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="25321-175">Todas as ações que são feitas usando o ficheiro inicial `.kube/config` são feitas usando uma conta de administração de cluster.</span><span class="sxs-lookup"><span data-stu-id="25321-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="25321-176">Pode agora experimentar vários comandos utilizando `kubectl` para verificar o estado do seu cluster.</span><span class="sxs-lookup"><span data-stu-id="25321-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="25321-177">Kubernetes tem o seu próprio modelo _ *Control de Acesso baseado em funções (RBAC)*\* que lhe permite criar definições de papéis finos e encadernações de papéis.</span><span class="sxs-lookup"><span data-stu-id="25321-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="25321-178">Esta é a forma preferível de controlar o acesso ao cluster em vez de distribuir permissões de administração de cluster.</span><span class="sxs-lookup"><span data-stu-id="25321-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="25321-179">Ligue os gasodutos Azure aos clusters de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="25321-180">Para ligar os Gasodutos Azure ao recém-implantado cluster Kubernetes, precisamos do seu ficheiro kube config ( `.kube/config` ) como explicado no passo anterior.</span><span class="sxs-lookup"><span data-stu-id="25321-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="25321-181">Ligue-se a um dos nós mestres do seu aglomerado kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="25321-182">Copie o conteúdo do `.kube/config` ficheiro.</span><span class="sxs-lookup"><span data-stu-id="25321-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="25321-183">Vá ao Azure DevOps > Project Settings > Conexões de Serviço para criar uma nova ligação de serviço "Kubernetes" (utilize KubeConfig como método de autenticação)</span><span class="sxs-lookup"><span data-stu-id="25321-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="25321-184">Os Gasodutos Azure (ou os seus agentes de construção) devem ter acesso à API de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="25321-185">Se houver uma ligação à Internet dos Gasodutos Azure ao clusetr Azure Stack Hub Kubernetes, terá de implantar um agente de construção de gasodutos Azure.</span><span class="sxs-lookup"><span data-stu-id="25321-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="25321-186">Ao implementar agentes auto-hospedados para a Azure Pipelines, pode implantar-se no Azure Stack Hub ou numa máquina com conectividade de rede a todos os pontos finais de gestão necessários.</span><span class="sxs-lookup"><span data-stu-id="25321-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="25321-187">Veja os detalhes aqui:</span><span class="sxs-lookup"><span data-stu-id="25321-187">See the details here:</span></span>

* <span data-ttu-id="25321-188">[Agentes da Azure Pipelines](/azure/devops/pipelines/agents/agents) no [Windows](/azure/devops/pipelines/agents/v2-windows) ou [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="25321-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="25321-189">A secção [de considerações de implementação de padrões (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) contém um fluxo de decisão que o ajuda a entender se deve usar agentes hospedados pela Microsoft ou agentes auto-hospedados:</span><span class="sxs-lookup"><span data-stu-id="25321-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="25321-190">[![fluxo de decisão agentes auto-hospedados](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="25321-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="25321-191">Nesta solução de amostra, a topologia inclui um agente de construção auto-hospedado em cada instância do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="25321-192">O agente pode aceder aos pontos finais de gestão do Hub Azure Stack e dos pontos finais do cluster Kubernetes API.</span><span class="sxs-lookup"><span data-stu-id="25321-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="25321-193">[![apenas tráfego de saída](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="25321-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="25321-194">Este design preenche um requisito regulamentar comum, que é ter apenas ligações de saída da solução de aplicação.</span><span class="sxs-lookup"><span data-stu-id="25321-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="25321-195">Configurar a monitorização</span><span class="sxs-lookup"><span data-stu-id="25321-195">Configure monitoring</span></span>

<span data-ttu-id="25321-196">Pode utilizar [o Azure Monitor](/azure/azure-monitor/) para obter recipientes para monitorizar os recipientes na solução.</span><span class="sxs-lookup"><span data-stu-id="25321-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="25321-197">Isto aponta o Azure Monitor para o cluster Kubernetes implantado pelo motor AKS no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="25321-198">Existem duas formas de ativar o Monitor Azure no seu cluster.</span><span class="sxs-lookup"><span data-stu-id="25321-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="25321-199">Ambas as formas requerem que você crie um espaço de trabalho Log Analytics em Azure.</span><span class="sxs-lookup"><span data-stu-id="25321-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="25321-200">[Método um](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa um gráfico de leme</span><span class="sxs-lookup"><span data-stu-id="25321-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="25321-201">[Método dois](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) como parte da especificação do cluster do motor AKS</span><span class="sxs-lookup"><span data-stu-id="25321-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="25321-202">Na topologia da amostra é utilizado o "Método um", que permite automatizar o processo e as atualizações podem ser instaladas mais facilmente.</span><span class="sxs-lookup"><span data-stu-id="25321-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="25321-203">Para o próximo passo, precisa de um espaço de trabalho Azure LogAnalytics (ID e Chave), `Helm` (versão 3) e `kubectl` na sua máquina.</span><span class="sxs-lookup"><span data-stu-id="25321-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="25321-204">Helm é um gestor de pacotes Kubernetes, disponível como binário que é executado em macOS, Windows e Linux.</span><span class="sxs-lookup"><span data-stu-id="25321-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="25321-205">Pode ser descarregado aqui: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm depende do ficheiro de configuração Kubernetes utilizado para o `kubectl` comando.</span><span class="sxs-lookup"><span data-stu-id="25321-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="25321-206">Este comando instalará o agente Azure Monitor no seu cluster Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="25321-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="25321-207">O Agente do Suite de Gestão de Operações (OMS) do seu cluster Kubernetes enviará dados de monitorização para o seu espaço de trabalho Azure Log Analytics (utilizando HTTPS de saída).</span><span class="sxs-lookup"><span data-stu-id="25321-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="25321-208">Agora pode utilizar o Azure Monitor para obter informações mais profundas sobre os seus clusters Kubernetes no Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="25321-209">Este design é uma forma poderosa de demonstrar o poder da análise que pode ser automaticamente implementado com os clusters da sua aplicação.</span><span class="sxs-lookup"><span data-stu-id="25321-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="25321-210">[![Aglomerados Azure Stack Hub no monitor Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="25321-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="25321-211">[![Detalhes do cluster do Monitor Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="25321-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="25321-212">Se o Azure Monitor não mostrar quaisquer dados do Azure Stack Hub, certifique-se de que seguiu cuidadosamente as instruções sobre [como adicionar AzureMonitor-Containers solução a um espaço de trabalho Azure Loganalytics.](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)</span><span class="sxs-lookup"><span data-stu-id="25321-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="25321-213">Implementar a aplicação</span><span class="sxs-lookup"><span data-stu-id="25321-213">Deploy the application</span></span>

<span data-ttu-id="25321-214">Antes de instalar a nossa aplicação de amostra, há mais um passo para configurar o controlador Ingress baseado em Nginx no nosso cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="25321-215">O controlador Ingress é usado como um equilibrador de carga de camada 7 para encaminhar o tráfego no nosso cluster com base no hospedeiro, caminho ou protocolo.</span><span class="sxs-lookup"><span data-stu-id="25321-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="25321-216">Nginx-ingress está disponível como um Cartão de Leme.</span><span class="sxs-lookup"><span data-stu-id="25321-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="25321-217">Para obter instruções detalhadas, consulte o [repositório Helm Chart GitHub](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="25321-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="25321-218">A nossa aplicação de amostra também é embalada como um Gráfico de Leme, como o [Agente de Monitorização Azure](#configure-monitoring) no passo anterior.</span><span class="sxs-lookup"><span data-stu-id="25321-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="25321-219">Como tal, é simples colocar a aplicação no nosso cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="25321-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="25321-220">Você pode encontrar os [ficheiros Helm Chart no repo do companheiro GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="25321-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="25321-221">A aplicação da amostra é uma aplicação de três níveis, implantada num cluster Kubernetes em cada uma das duas instâncias do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="25321-222">A aplicação utiliza uma base de dados MongoDB.</span><span class="sxs-lookup"><span data-stu-id="25321-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="25321-223">Pode aprender mais sobre como obter os dados replicados em várias instâncias nas considerações de [dados e armazenamento de padrões.](pattern-highly-available-kubernetes.md#data-and-storage-considerations)</span><span class="sxs-lookup"><span data-stu-id="25321-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="25321-224">Depois de implementar o Gráfico Helm para a aplicação, verá os três níveis da sua aplicação representados como implementações e conjuntos imponentes (para a base de dados) com uma única cápsula:</span><span class="sxs-lookup"><span data-stu-id="25321-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="25321-225">Nos serviços, o lado encontrará o Controlador Ingress baseado em Nginx e o seu endereço IP público:</span><span class="sxs-lookup"><span data-stu-id="25321-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="25321-226">O endereço "IP Externo" é o nosso "ponto final de aplicação".</span><span class="sxs-lookup"><span data-stu-id="25321-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="25321-227">É assim que os utilizadores se ligarão para abrir a aplicação e também serão usados como ponto final para o nosso próximo passo [Configure Traffic Manager](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="25321-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="25321-228">Autoescala a aplicação</span><span class="sxs-lookup"><span data-stu-id="25321-228">Autoscale the application</span></span>
<span data-ttu-id="25321-229">Pode configurar opcionalmente o [Autoscaler horizontal do pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) para escalar para cima ou para baixo com base em determinadas métricas como a utilização do CPU.</span><span class="sxs-lookup"><span data-stu-id="25321-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="25321-230">O seguinte comando criará um Pod Autoscaler horizontal que mantém 1 a 10 réplicas dos Pods controlados pela implementação web de classificações.</span><span class="sxs-lookup"><span data-stu-id="25321-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="25321-231">O HPA aumentará e diminuirá o número de réplicas (através da implantação) para manter uma utilização média do CPU em todos os Pods de 80%.</span><span class="sxs-lookup"><span data-stu-id="25321-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="25321-232">Pode verificar o estado atual do autoescalador executando:</span><span class="sxs-lookup"><span data-stu-id="25321-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="25321-233">Configurar o Gestor de Tráfego</span><span class="sxs-lookup"><span data-stu-id="25321-233">Configure Traffic Manager</span></span>

<span data-ttu-id="25321-234">Para distribuir o tráfego entre duas (ou mais) implementações da aplicação, utilizaremos [o Gestor de Tráfego Azure.](/azure/traffic-manager/traffic-manager-overview)</span><span class="sxs-lookup"><span data-stu-id="25321-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="25321-235">Azure Traffic Manager é um equilibrador de carga baseado em DNS em Azure.</span><span class="sxs-lookup"><span data-stu-id="25321-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="25321-236">O Gestor de Tráfego utiliza o DNS para direcionar os pedidos dos clientes para o ponto final de serviço mais adequado, com base num método de encaminhamento de tráfego e na saúde dos pontos finais.</span><span class="sxs-lookup"><span data-stu-id="25321-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="25321-237">Em vez de utilizar o Azure Traffic Manager, também pode utilizar outras soluções globais de equilíbrio de carga hospedadas no local.</span><span class="sxs-lookup"><span data-stu-id="25321-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="25321-238">No cenário da amostra, usaremos o Azure Traffic Manager para distribuir o tráfego entre duas instâncias da nossa aplicação.</span><span class="sxs-lookup"><span data-stu-id="25321-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="25321-239">Podem correr em instâncias do Azure Stack Hub nos mesmos locais ou locais diferentes:</span><span class="sxs-lookup"><span data-stu-id="25321-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![gestor de tráfego no local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="25321-241">Em Azure, configuramos o Gestor de Tráfego para apontar para as duas instâncias diferentes da nossa aplicação:</span><span class="sxs-lookup"><span data-stu-id="25321-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="25321-242">[![Perfil de ponto final TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="25321-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="25321-243">Como podem ver, os dois pontos finais apontam para as duas instâncias da aplicação implantada da [secção anterior.](#deploy-the-application)</span><span class="sxs-lookup"><span data-stu-id="25321-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="25321-244">Neste ponto:</span><span class="sxs-lookup"><span data-stu-id="25321-244">At this point:</span></span>
- <span data-ttu-id="25321-245">A infraestrutura Kubernetes foi criada, incluindo um Controlador ingress.</span><span class="sxs-lookup"><span data-stu-id="25321-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="25321-246">Os clusters foram implantados em duas instâncias do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="25321-247">A monitorização foi configurada.</span><span class="sxs-lookup"><span data-stu-id="25321-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="25321-248">O Azure Traffic Manager carregará o tráfego de equilíbrio através das duas instâncias do Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="25321-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="25321-249">Além desta infraestrutura, a aplicação de três níveis da amostra foi implementada de forma automatizada utilizando o Helm Charts.</span><span class="sxs-lookup"><span data-stu-id="25321-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="25321-250">A solução deve agora ser acessível e acessível aos utilizadores!</span><span class="sxs-lookup"><span data-stu-id="25321-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="25321-251">Há também algumas considerações operacionais pós-implantação que merecem ser discutidas, que são abordadas nas duas secções seguintes.</span><span class="sxs-lookup"><span data-stu-id="25321-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="25321-252">Atualizar Kubernetes</span><span class="sxs-lookup"><span data-stu-id="25321-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="25321-253">Considere os seguintes tópicos ao atualizar o cluster Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="25321-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="25321-254">A atualização de um cluster Kubernetes é uma operação complexa do Dia 2 que pode ser feita com motor AKS.</span><span class="sxs-lookup"><span data-stu-id="25321-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="25321-255">Para obter mais informações, consulte [upgrade de um cluster Kubernetes no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="25321-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="25321-256">O Motor AKS permite-lhe atualizar clusters para as versões de imagem mais recentes de Kubernetes e os bases da imagem do SO.</span><span class="sxs-lookup"><span data-stu-id="25321-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="25321-257">Para obter mais informações, consulte [Steps para atualizar para uma versão mais recente de Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="25321-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="25321-258">Também pode atualizar apenas os nós de subserção para as versões de imagem oss mais recentes.</span><span class="sxs-lookup"><span data-stu-id="25321-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="25321-259">Para obter mais informações, consulte [Passos para atualizar apenas a imagem do SO](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="25321-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="25321-260">As imagens mais recentes do SISTEMA de Base contêm atualizações de segurança e kernel.</span><span class="sxs-lookup"><span data-stu-id="25321-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="25321-261">É da responsabilidade do operador do cluster monitorizar a disponibilidade de versões mais recentes de Kubernetes e IMAGENS OS.</span><span class="sxs-lookup"><span data-stu-id="25321-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="25321-262">O operador deve planear e executar estas atualizações utilizando o motor AKS.</span><span class="sxs-lookup"><span data-stu-id="25321-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="25321-263">As imagens base OS devem ser descarregadas do Azure Stack Hub Marketplace pelo Azure Stack Hub Operator.</span><span class="sxs-lookup"><span data-stu-id="25321-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="25321-264">Kubernetes de escala</span><span class="sxs-lookup"><span data-stu-id="25321-264">Scale Kubernetes</span></span>

<span data-ttu-id="25321-265">Scale é outra operação do Dia 2 que pode ser orquestrada usando o motor AKS.</span><span class="sxs-lookup"><span data-stu-id="25321-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="25321-266">O comando de escala reutiliza o seu ficheiro de configuração do cluster (apimodel.jsligado) no diretório de saída, como entrada para uma nova implantação do Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="25321-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="25321-267">O Motor AKS executa a operação de escala contra um conjunto de agentes específicos.</span><span class="sxs-lookup"><span data-stu-id="25321-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="25321-268">Quando a operação de escala estiver concluída, o Motor AKS atualiza a definição de cluster nesse mesmo apimodel.jsem ficheiro.</span><span class="sxs-lookup"><span data-stu-id="25321-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="25321-269">A definição de cluster reflete a nova contagem de nós de forma a refletir a configuração atualizada e atual do cluster.</span><span class="sxs-lookup"><span data-stu-id="25321-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="25321-270">Escalar um cluster Kubernetes no Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="25321-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="25321-271">Passos seguintes</span><span class="sxs-lookup"><span data-stu-id="25321-271">Next steps</span></span>

- <span data-ttu-id="25321-272">Saiba mais sobre [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="25321-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="25321-273">Reveja e proponha melhorias [ao código para esta amostra no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="25321-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>