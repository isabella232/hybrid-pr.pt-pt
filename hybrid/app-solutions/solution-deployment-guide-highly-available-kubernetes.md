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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Implementar um cluster Kubernetes de alta disponibilidade no Azure Stack Hub

Este artigo irá mostrar-lhe como construir um ambiente de cluster Kubernetes altamente disponível, implantado em várias instâncias do Azure Stack Hub, em diferentes locais físicos.

Neste guia de implementação de soluções, aprende-se a:

> [!div class="checklist"]
> - Descarregue e prepare o motor AKS
> - Ligue-se ao VM do Ajudante do Motor AKS
> - Implementar um cluster Kubernetes
> - Ligue-se ao cluster Kubernetes
> - Ligue os gasodutos Azure ao cluster Kubernetes
> - Configurar a monitorização
> - Implementar aplicação
> - Aplicação de escala automática
> - Configurar o Gestor de Tráfego
> - Atualizar Kubernetes
> - Kubernetes de escala

> [!Tip]  
> ![Pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Antes de começar com este guia de implantação, certifique-se de que:

- Reveja o artigo [de padrão de cluster kubernetes de alta disponibilidade.](pattern-highly-available-kubernetes.md)
- Reveja o conteúdo do [repositório GitHub,](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)que contém ativos adicionais referenciados neste artigo.
- Ter uma conta que possa aceder ao portal de [utilizadores do Azure Stack Hub,](/azure-stack/user/azure-stack-use-portal)com pelo menos [permissões de "contribuinte".](/azure-stack/user/azure-stack-manage-permissions)

## <a name="download-and-prepare-aks-engine"></a>Baixar e preparar motor AKS

O Motor AKS é um binário que pode ser usado a partir de qualquer anfitrião Windows ou Linux que possa chegar aos pontos finais do Azure Stack Hub Azure Resource Manager. Este guia descreve a implementação de um novo VM Linux (ou Windows) no Azure Stack Hub. Será usado mais tarde quando o motor AKS implementar os clusters Kubernetes.

> [!NOTE]
> Também pode utilizar um Windows ou Linux VM existente para implantar um cluster Kubernetes no Azure Stack Hub utilizando o Motor AKS.

O processo passo a passo e os requisitos para o motor AKS estão documentados aqui:

* [Instale o motor AKS no Linux no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (ou utilizando o [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

O Motor AKS é uma ferramenta auxiliar para implantar e operar clusters Kubernetes (em Azure e Azure Stack Hub).

Os detalhes e diferenças do motor AKS no Azure Stack Hub são descritos aqui:

* [O que é o motor AKS no Azure Stack Hub?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Motor AKS no Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (no GitHub)

O ambiente da amostra utilizará o Terraform para automatizar a implantação do VM do motor AKS. Você pode encontrar os [detalhes e código no companheiro GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

O resultado deste passo é um novo grupo de recursos no Azure Stack Hub que contém o VM do ajudante do motor AKS e recursos conexos:

![Recursos VM do motor AKS no Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Se tiver de implantar o Motor AKS num ambiente desligado, [reveja as instâncias do Hub de Pilha de Azure para](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) saber mais.

No próximo passo, usaremos o recém-implantado VM do motor AKS para implantar um cluster Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Ligue-se ao VM ajudante do motor AKS

Primeiro, deve ligar-se ao VM do ajudante do motor AKS anteriormente criado.

O VM deve ter um Endereço IP Público e deve estar acessível via SSH (Porta 22/TCP).

![Página geral do motor AKS VM](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Pode utilizar uma ferramenta à sua escolha como MobaXterm, puTTY ou PowerShell no Windows 10 para se ligar a um Linux VM utilizando SSH.

```console
ssh <username>@<ipaddress>
```

Depois de ligar, executar o comando `aks-engine` . Vá às [versões suportadas do motor AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) para saber mais sobre as versões AKS Engine e Kubernetes.

![exemplo da linha de comando do motor aks](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Implementar um cluster Kubernetes

O próprio VM, ajudante do motor AKS, ainda não criou um cluster Kubernetes no nosso Azure Stack Hub. A criação do cluster é a primeira ação a tomar no VM do ajudante do motor AKS.

O processo passo a passo é documentado aqui:

* [Implementar um cluster Kubernetes com o motor AKS no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

O resultado final do `aks-engine deploy` comando e dos preparativos nos passos anteriores é um cluster Kubernetes totalmente apresentado implantado no espaço de inquilino da primeira instância do Azure Stack Hub. O cluster em si é composto por componentes Azure IaaS como VMs, equilibradores de carga, VNets, discos, etc.

![Cluster IaaS componentes Azure Stack Hub portal](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Equilibrador de carga Azure (K8s API Endpoint)
2) Nós operários (Piscina do Agente)
3) Mestre Nosdes

O aglomerado está a funcionar e no próximo passo vamos ligar-nos a ele.

## <a name="connect-to-the-kubernetes-cluster"></a>Ligue-se ao cluster Kubernetes

Pode agora ligar-se ao cluster Kubernetes anteriormente criado, quer via SSH (utilizando a chave SSH especificada como parte da implantação) quer via `kubectl` (recomendada). A ferramenta de linha de comando Kubernetes `kubectl` está disponível para Windows, Linux e macOS [aqui.](https://kubernetes.io/docs/tasks/tools/install-kubectl/) Já está pré-instalado e configurado nos nós mestres do nosso aglomerado.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Execute kubectl no nó mestre](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Não é recomendado usar o nó mestre como uma caixa de salto para tarefas administrativas. A `kubectl` configuração é armazenada `.kube/config` no(s) nó(s) principal, bem como no VM do motor AKS. Pode copiar a configuração para uma máquina de administração com conectividade ao cluster Kubernetes e utilizar o `kubectl` comando lá. O `.kube/config` ficheiro também é usado mais tarde para configurar uma ligação de serviço em Azure Pipelines.

> [!IMPORTANT]
> Mantenha estes ficheiros seguros porque contêm as credenciais para o seu cluster Kubernetes. Um intruso com acesso ao ficheiro tem informações suficientes para obter acesso ao administrador. Todas as ações que são feitas usando o ficheiro inicial `.kube/config` são feitas usando uma conta de administração de cluster.

Pode agora experimentar vários comandos utilizando `kubectl` para verificar o estado do seu cluster.

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
> Kubernetes tem o seu próprio modelo _ *Control de Acesso baseado em funções (RBAC)** que lhe permite criar definições de papéis finos e encadernações de papéis. Esta é a forma preferível de controlar o acesso ao cluster em vez de distribuir permissões de administração de cluster.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Ligue os gasodutos Azure aos clusters de Kubernetes

Para ligar os Gasodutos Azure ao recém-implantado cluster Kubernetes, precisamos do seu ficheiro kube config ( `.kube/config` ) como explicado no passo anterior.

* Ligue-se a um dos nós mestres do seu aglomerado kubernetes.
* Copie o conteúdo do `.kube/config` ficheiro.
* Vá ao Azure DevOps > Project Settings > Conexões de Serviço para criar uma nova ligação de serviço "Kubernetes" (utilize KubeConfig como método de autenticação)

> [!IMPORTANT]
> Os Gasodutos Azure (ou os seus agentes de construção) devem ter acesso à API de Kubernetes. Se houver uma ligação à Internet dos Gasodutos Azure ao clusetr Azure Stack Hub Kubernetes, terá de implantar um agente de construção de gasodutos Azure.

Ao implementar agentes auto-hospedados para a Azure Pipelines, pode implantar-se no Azure Stack Hub ou numa máquina com conectividade de rede a todos os pontos finais de gestão necessários. Veja os detalhes aqui:

* [Agentes da Azure Pipelines](/azure/devops/pipelines/agents/agents) no [Windows](/azure/devops/pipelines/agents/v2-windows) ou [Linux](/azure/devops/pipelines/agents/v2-linux)

A secção [de considerações de implementação de padrões (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) contém um fluxo de decisão que o ajuda a entender se deve usar agentes hospedados pela Microsoft ou agentes auto-hospedados:

[![fluxo de decisão agentes auto-hospedados](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

Nesta solução de amostra, a topologia inclui um agente de construção auto-hospedado em cada instância do Azure Stack Hub. O agente pode aceder aos pontos finais de gestão do Hub Azure Stack e dos pontos finais do cluster Kubernetes API.

[![apenas tráfego de saída](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Este design preenche um requisito regulamentar comum, que é ter apenas ligações de saída da solução de aplicação.

## <a name="configure-monitoring"></a>Configurar a monitorização

Pode utilizar [o Azure Monitor](/azure/azure-monitor/) para obter recipientes para monitorizar os recipientes na solução. Isto aponta o Azure Monitor para o cluster Kubernetes implantado pelo motor AKS no Azure Stack Hub.

Existem duas formas de ativar o Monitor Azure no seu cluster. Ambas as formas requerem que você crie um espaço de trabalho Log Analytics em Azure.

* [Método um](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa um gráfico de leme
* [Método dois](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) como parte da especificação do cluster do motor AKS

Na topologia da amostra é utilizado o "Método um", que permite automatizar o processo e as atualizações podem ser instaladas mais facilmente.

Para o próximo passo, precisa de um espaço de trabalho Azure LogAnalytics (ID e Chave), `Helm` (versão 3) e `kubectl` na sua máquina.

Helm é um gestor de pacotes Kubernetes, disponível como binário que é executado em macOS, Windows e Linux. Pode ser descarregado aqui: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm depende do ficheiro de configuração Kubernetes utilizado para o `kubectl` comando.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Este comando instalará o agente Azure Monitor no seu cluster Kubernetes:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

O Agente do Suite de Gestão de Operações (OMS) do seu cluster Kubernetes enviará dados de monitorização para o seu espaço de trabalho Azure Log Analytics (utilizando HTTPS de saída). Agora pode utilizar o Azure Monitor para obter informações mais profundas sobre os seus clusters Kubernetes no Azure Stack Hub. Este design é uma forma poderosa de demonstrar o poder da análise que pode ser automaticamente implementado com os clusters da sua aplicação.

[![Aglomerados Azure Stack Hub no monitor Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Detalhes do cluster do Monitor Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Se o Azure Monitor não mostrar quaisquer dados do Azure Stack Hub, certifique-se de que seguiu cuidadosamente as instruções sobre [como adicionar AzureMonitor-Containers solução a um espaço de trabalho Azure Loganalytics.](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)

## <a name="deploy-the-application"></a>Implementar a aplicação

Antes de instalar a nossa aplicação de amostra, há mais um passo para configurar o controlador Ingress baseado em Nginx no nosso cluster Kubernetes. O controlador Ingress é usado como um equilibrador de carga de camada 7 para encaminhar o tráfego no nosso cluster com base no hospedeiro, caminho ou protocolo. Nginx-ingress está disponível como um Cartão de Leme. Para obter instruções detalhadas, consulte o [repositório Helm Chart GitHub](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

A nossa aplicação de amostra também é embalada como um Gráfico de Leme, como o [Agente de Monitorização Azure](#configure-monitoring) no passo anterior. Como tal, é simples colocar a aplicação no nosso cluster Kubernetes. Você pode encontrar os [ficheiros Helm Chart no repo do companheiro GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

A aplicação da amostra é uma aplicação de três níveis, implantada num cluster Kubernetes em cada uma das duas instâncias do Azure Stack Hub. A aplicação utiliza uma base de dados MongoDB. Pode aprender mais sobre como obter os dados replicados em várias instâncias nas considerações de [dados e armazenamento de padrões.](pattern-highly-available-kubernetes.md#data-and-storage-considerations)

Depois de implementar o Gráfico Helm para a aplicação, verá os três níveis da sua aplicação representados como implementações e conjuntos imponentes (para a base de dados) com uma única cápsula:

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

Nos serviços, o lado encontrará o Controlador Ingress baseado em Nginx e o seu endereço IP público:

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

O endereço "IP Externo" é o nosso "ponto final de aplicação". É assim que os utilizadores se ligarão para abrir a aplicação e também serão usados como ponto final para o nosso próximo passo [Configure Traffic Manager](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Autoescala a aplicação
Pode configurar opcionalmente o [Autoscaler horizontal do pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) para escalar para cima ou para baixo com base em determinadas métricas como a utilização do CPU. O seguinte comando criará um Pod Autoscaler horizontal que mantém 1 a 10 réplicas dos Pods controlados pela implementação web de classificações. O HPA aumentará e diminuirá o número de réplicas (através da implantação) para manter uma utilização média do CPU em todos os Pods de 80%.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Pode verificar o estado atual do autoescalador executando:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Configurar o Gestor de Tráfego

Para distribuir o tráfego entre duas (ou mais) implementações da aplicação, utilizaremos [o Gestor de Tráfego Azure.](/azure/traffic-manager/traffic-manager-overview) Azure Traffic Manager é um equilibrador de carga baseado em DNS em Azure.

> [!NOTE]
> O Gestor de Tráfego utiliza o DNS para direcionar os pedidos dos clientes para o ponto final de serviço mais adequado, com base num método de encaminhamento de tráfego e na saúde dos pontos finais.

Em vez de utilizar o Azure Traffic Manager, também pode utilizar outras soluções globais de equilíbrio de carga hospedadas no local. No cenário da amostra, usaremos o Azure Traffic Manager para distribuir o tráfego entre duas instâncias da nossa aplicação. Podem correr em instâncias do Azure Stack Hub nos mesmos locais ou locais diferentes:

![gestor de tráfego no local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Em Azure, configuramos o Gestor de Tráfego para apontar para as duas instâncias diferentes da nossa aplicação:

[![Perfil de ponto final TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Como podem ver, os dois pontos finais apontam para as duas instâncias da aplicação implantada da [secção anterior.](#deploy-the-application)

Neste ponto:
- A infraestrutura Kubernetes foi criada, incluindo um Controlador ingress.
- Os clusters foram implantados em duas instâncias do Azure Stack Hub.
- A monitorização foi configurada.
- O Azure Traffic Manager carregará o tráfego de equilíbrio através das duas instâncias do Azure Stack Hub.
- Além desta infraestrutura, a aplicação de três níveis da amostra foi implementada de forma automatizada utilizando o Helm Charts. 

A solução deve agora ser acessível e acessível aos utilizadores!

Há também algumas considerações operacionais pós-implantação que merecem ser discutidas, que são abordadas nas duas secções seguintes.

## <a name="upgrade-kubernetes"></a>Atualizar Kubernetes

Considere os seguintes tópicos ao atualizar o cluster Kubernetes:

- A atualização de um cluster Kubernetes é uma operação complexa do Dia 2 que pode ser feita com motor AKS. Para obter mais informações, consulte [upgrade de um cluster Kubernetes no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- O Motor AKS permite-lhe atualizar clusters para as versões de imagem mais recentes de Kubernetes e os bases da imagem do SO. Para obter mais informações, consulte [Steps para atualizar para uma versão mais recente de Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Também pode atualizar apenas os nós de subserção para as versões de imagem oss mais recentes. Para obter mais informações, consulte [Passos para atualizar apenas a imagem do SO](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

As imagens mais recentes do SISTEMA de Base contêm atualizações de segurança e kernel. É da responsabilidade do operador do cluster monitorizar a disponibilidade de versões mais recentes de Kubernetes e IMAGENS OS. O operador deve planear e executar estas atualizações utilizando o motor AKS. As imagens base OS devem ser descarregadas do Azure Stack Hub Marketplace pelo Azure Stack Hub Operator.

## <a name="scale-kubernetes"></a>Kubernetes de escala

Scale é outra operação do Dia 2 que pode ser orquestrada usando o motor AKS.

O comando de escala reutiliza o seu ficheiro de configuração do cluster (apimodel.jsligado) no diretório de saída, como entrada para uma nova implantação do Azure Resource Manager. O Motor AKS executa a operação de escala contra um conjunto de agentes específicos. Quando a operação de escala estiver concluída, o Motor AKS atualiza a definição de cluster nesse mesmo apimodel.jsem ficheiro. A definição de cluster reflete a nova contagem de nós de forma a refletir a configuração atualizada e atual do cluster.

- [Escalar um cluster Kubernetes no Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Passos seguintes

- Saiba mais sobre [considerações de design de aplicativos Híbridos](overview-app-design-considerations.md)
- Reveja e proponha melhorias [ao código para esta amostra no GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).