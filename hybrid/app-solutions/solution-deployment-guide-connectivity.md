---
title: Configurar conectividade híbrida em nuvem em Azure e Azure Stack Hub
description: Aprenda a configurar a conectividade híbrida em nuvem utilizando o Azure e o Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4480f51b03082f2a0cbb7f2f213e05b7bf488646
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895384"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Configurar conectividade híbrida em nuvem usando Azure e Azure Stack Hub

Você pode aceder a recursos com segurança no Global Azure e Azure Stack Hub usando o padrão de conectividade híbrida.

Nesta solução, você construirá um ambiente de amostra para:

> [!div class="checklist"]
> - Mantenha os dados no local para satisfazer os requisitos de privacidade ou regulamentação, mas mantenha o acesso aos recursos globais da Azure.
> - Mantenha um sistema legado enquanto utiliza a implementação de aplicações em escala de nuvem e recursos no Azure global.

> [!Tip]  
> ![Diagrama de pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

## <a name="prerequisites"></a>Pré-requisitos

Alguns componentes são necessários para construir uma implantação híbrida de conectividade. Alguns destes componentes demoram tempo a preparar-se, por isso planeiem em conformidade.

### <a name="azure"></a>Azure

- Se não tiver uma subscrição do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.
- Crie uma [aplicação web](/aspnet/core/tutorials/publish-to-azure-webapp-using-vs) em Azure. Tome nota do URL da aplicação web porque vai precisar dele na solução.

### <a name="azure-stack-hub"></a>Azure Stack Hub

Um parceiro Azure OEM/hardware pode implementar um Azure Stack Hub de produção, e todos os utilizadores podem implementar um Kit de Desenvolvimento de Pilhas Azure (ASDK).

- Utilize a sua produção Azure Stack Hub ou implemente o ASDK.
   >[!Note]
   >A implantação da ASDK pode demorar até 7 horas, por isso planeie em conformidade.

- Implementar serviços paaS do Serviço de [Aplicações](/azure-stack/operator/azure-stack-app-service-deploy) para o Azure Stack Hub.
- [Crie planos e ofertas](/azure-stack/operator/service-plan-offer-subscription-overview) no ambiente Azure Stack Hub.
- [Crie subscrição de inquilino](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) dentro do ambiente Azure Stack Hub.

### <a name="azure-stack-hub-components"></a>Componentes do Azure Stack Hub

Um operador do Azure Stack Hub deve implementar o Serviço de Aplicações, criar planos e ofertas, criar uma subscrição de inquilino e adicionar a imagem do Windows Server 2016. Se já tiver estes componentes, certifique-se de que cumprem os requisitos antes de iniciar esta solução.

Este exemplo de solução pressupõe que você tem algum conhecimento básico de Azure e Azure Stack Hub. Para saber mais antes de iniciar a solução, leia os seguintes artigos:

- [Introdução ao Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Conceitos de chave de hub de pilha de Azure](/azure-stack/operator/azure-stack-overview)

### <a name="before-you-begin"></a>Antes de começar

Verifique se cumpre os seguintes critérios antes de começar a configurar a conectividade híbrida em nuvem:

- Precisa de um endereço IPv4 público virado para o exterior para o seu dispositivo VPN. Este endereço IP não pode ser localizado atrás de um NAT (Tradução de Endereços de Rede).
- Todos os recursos são mobilizados na mesma região/local.

#### <a name="solution-example-values"></a>Valores de exemplo de solução

Os exemplos desta solução utilizam os seguintes valores. Pode utilizar estes valores para criar um ambiente de teste ou encaminhá-los para uma melhor compreensão dos exemplos. Para obter mais informações sobre as definições de gateway VPN, consulte [sobre as definições do gateway VPN](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Especificações de ligação:

- **Tipo VPN**: baseado em rotas
- Tipo de **ligação**: site-to-site (IPsec)
- **Tipo gateway**: VPN
- **Nome de ligação azul**: Azure-Gateway-AzureStack-S2SGateway (o portal preencherá automaticamente este valor)
- **Nome de conexão Azure Stack Hub**: AzureStack-Gateway-Azure-S2SGateway (o portal preencherá automaticamente este valor)
- **Chave partilhada**: qualquer compatível com hardware VPN, com valores correspondentes em ambos os lados da ligação
- **Subscrição**: qualquer subscrição preferida
- **Grupo de recursos**: Test-Infra

Endereços IP de rede e sub-rede:

| Ligação do hub Azure/Azure Stack | Name | Sub-rede | Endereço IP |
|---|---|---|---|
| Azure vNet | AplicaçãovNet<br>10.100.102.9/23 | AplicaçõesSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Azure Stack Hub vNet | AplicaçãovNet<br>10.100.100.0/23 | AplicaçõesSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Gateway de rede virtual Azure | Azure-Gateway |  |  |
| Gateway de rede virtual Azure Stack Hub | AzureStack-Gateway |  |  |
| IP Público do Azure | Azure-GatewayPublicIP |  | Determinado na criação |
| Azure Stack Hub Public IP | AzureStack-GatewayPublicIP |  | Determinado na criação |
| Gateway de rede local de Azure | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Valor IP público do Hub Azure Stack |
| Gateway de rede local Azure Stack Hub | Azure-S2SGateway<br>10.100.102.0/23 |  | Valor IP Público Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Criar uma rede virtual no Global Azure e Azure Stack Hub

Utilize os seguintes passos para criar uma rede virtual utilizando o portal. Pode utilizar estes [valores de exemplo](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) se estiver a usar este artigo como única solução. Se estiver a usar este artigo para configurar um ambiente de produção, substitua as definições de exemplo pelos seus próprios valores.

> [!IMPORTANT]
> Deve certificar-se de que não existe uma sobreposição de endereços IP nos espaços de endereçoS Azure Stack Hub vNet.

Para criar um vNet em Azure:

1. Utilize o seu navegador para ligar ao [portal Azure](https://portal.azure.com/) e iniciar sação com a sua conta Azure.
2. Selecione **Criar um recurso**. No campo **'Pesquisar no mercado',** insira 'rede virtual'. Selecione **a rede Virtual** a partir dos resultados.
3. A partir da lista **de modelos de implementação,** selecione **Resource Manager** e, em seguida, selecione **Criar**.
4. Na **criação de rede virtual,** configurar as definições VNet. Os nomes dos campos necessários são prefixados com um asterisco vermelho.  Quando introduz um valor válido, o asterisco muda para uma marca de verificação verde.

Para criar um vNet no Azure Stack Hub:

1. Repita os passos acima (1-4) utilizando o portal de **inquilinos** Azure Stack Hub .

## <a name="add-a-gateway-subnet"></a>Adicionar uma sub-rede do gateway

Antes de ligar a sua rede virtual a um gateway, tem de criar a sub-rede gateway para a rede virtual a que pretende ligar. Os serviços de gateway utilizam os endereços IP que especifica na sub-rede gateway.

No [portal Azure,](https://portal.azure.com/)navegue para a rede virtual Do Gestor de Recursos onde pretende criar um gateway de rede virtual.

1. Selecione o vNet para abrir a página **de rede Virtual.**
2. Em **DEFINIÇÕES,** selecione **Subnetas**.
3. Na página **Subnetas,** selecione **+Gateway sub-rede** para abrir a página **de sub-redes Add.**

    ![Adicionar sub-rede gateway](media/solution-deployment-guide-connectivity/image4.png)

4. O **nome** da sub-rede é automaticamente preenchido com o valor 'GatewaySubnet'. Este valor é necessário para que a Azure reconheça a sub-rede como sub-rede de gateway.
5. Altere os valores **de gama de endereços** fornecidos para corresponder aos seus requisitos de configuração e, em seguida, selecione **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Criar um Gateway de rede virtual em Azure e Azure Stack

Utilize os seguintes passos para criar uma porta de entrada de rede virtual em Azure.

1. No lado esquerdo da página do portal, selecione **+** e introduza 'gateway de rede virtual' no campo de pesquisa.
2. Em **Resultados**, selecione **Virtual network gateway**.
3. No **gateway de rede virtual**, selecione **Criar** para abrir a página de gateway de **rede virtual** Create.
4. No **Portal de Rede Virtual ,** especifique os valores do seu gateway de rede utilizando os **nossos valores de exemplo Tutorial**. Incluir os seguintes valores adicionais:

   - **SKU**: básico
   - **Rede Virtual**: Selecione a rede virtual que criou anteriormente. A sub-rede gateway que criou é selecionada automaticamente.
   - **Primeira Configuração IP**: O IP público do seu gateway.
     - Selecione **Criar configuração IP gateway**, que o leva à página de **endereços IP público** escolha.
     - Selecione **+Crie novo** para abrir a página **de endereços IP público.**
     - Insira um **Nome** para o seu endereço IP público. Deixe o SKU como **Básico** e, em seguida, selecione **OK** para guardar as suas alterações.

       > [!Note]
       > Atualmente, o VPN Gateway apenas suporta a atribuição de endereços IP dinâmicos públicos. No entanto, isto não significa que o endereço IP mude depois de ser atribuído ao seu gateway VPN. A única vez que o endereço IP público muda é quando o gateway é eliminado e recriado. Redimensionamento, reposição ou outras melhorias/manutenção interna para o seu gateway VPN não alterem o endereço IP.

5. Verifique as definições do gateway.
6. Selecione **Criar** para criar o gateway VPN. As definições do gateway são validadas e o azulejo "Implementar o gateway de rede virtual" é mostrado no seu painel de instrumentos.

   >[!Note]
   >A criação de um gateway pode demorar até 45 minutos. Poderá ter de atualizar a página do portal para ver o estado concluído.

    Após a criação do gateway, pode ver o endereço IP que lhe foi atribuído olhando para a rede virtual no portal. O gateway aparece como um dispositivo ligado. Para obter mais informações sobre o gateway, selecione o dispositivo.

7. Repita os passos anteriores (1-5) na sua implantação do Azure Stack Hub.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Crie o portal de rede local em Azure e Azure Stack Hub

O gateway de rede local refere-se normalmente à sua localização no local. Você dá ao site um nome que Azure ou Azure Stack Hub podem consultar e, em seguida, especificar:

- O endereço IP do dispositivo VPN no local para o qual está a criar uma ligação.
- Os prefixos do endereço IP que serão encaminhados através da porta de entrada VPN para o dispositivo VPN. Os prefixos do endereço que especificar são os que estão localizados na sua rede no local.

  >[!Note]
  >Se a sua rede no local mudar ou precisar de alterar o endereço IP público para o dispositivo VPN, poderá atualizar estes valores mais tarde.

1. No portal, selecione **+Criar um recurso.**
2. Na caixa de pesquisa, insira **o gateway de rede local** e, em seguida, selecione **Enter** para pesquisar. Uma lista de resultados será exibida.
3. Selecione **O gateway de rede local** e, em seguida, selecione **Criar** para abrir a página de gateway de **rede local.**
4. No **Portal da rede local,** especifique os valores do seu gateway de rede local utilizando os **nossos valores de exemplo Tutorial**. Incluir os seguintes valores adicionais:

    - **Endereço IP**: O endereço IP público do dispositivo VPN a que pretende que o Azure ou o Azure Stack Hub se conectem. Especifique um endereço IP público válido que não esteja por trás de um NAT para que o Azure possa chegar ao endereço. Se não tiver o endereço IP neste momento, pode utilizar um valor do exemplo como espaço reservado. Terá de voltar e substituir o espaço reservado pelo endereço IP público do seu dispositivo VPN. O Azure não pode ligar-se ao dispositivo até fornecer um endereço válido.
    - **Espaço endereço**: o intervalo de endereços para a rede que esta rede local representa. Pode adicionar vários intervalos de espaço de endereços. Certifique-se de que os intervalos especificados não se sobrepõem a gamas de outras redes a que pretende ligar. O Azure irá encaminhar o intervalo de endereços que especificou no endereço IP do dispositivo VPN no local. Use os seus próprios valores se quiser ligar-se ao seu site no local, e não um valor de exemplo.
    - **Configurar as definições de BGP**: Utilize apenas quando configurar o BGP. Caso contrário, não selecione esta opção.
    - **Assinatura**: Verifique se a subscrição correta está a aparecer.
    - **Grupo de Recursos**: Selecione o grupo de recursos que pretende utilizar. Pode criar um novo grupo de recursos ou selecionar um que já criou.
    - **Localização**: Selecione a localização em que este objeto será criado. Pode querer selecionar o mesmo local em que o seu VNet reside, mas não é obrigado a fazê-lo.
5. Quando terminar de especificar os valores necessários, **selecione Criar** para criar o gateway de rede local.
6. Repita estes passos (1-5) na sua implantação do Azure Stack Hub.

## <a name="configure-your-connection"></a>Configure a sua ligação

As ligações site-to-site a uma rede no local requerem um dispositivo VPN. O dispositivo VPN que configura é referido como uma ligação. Para configurar a sua ligação, precisa:

- Uma chave partilhada. Esta chave é a mesma chave partilhada que especifica ao criar a sua ligação VPN site-to-site. Nos nossos exemplos, iremos utilizar uma chave partilhada básica. Deve gerar uma chave mais complexa para utilizar.
- O endereço IP público do seu portal de rede virtual. Pode ver o endereço IP público através do portal do Azure, do PowerShell ou da CLI. Para encontrar o endereço IP público do seu gateway VPN utilizando o portal Azure, vá para gateways de rede virtuais e, em seguida, selecione o nome do seu gateway.

Utilize os seguintes passos para criar uma ligação VPN site-to-site entre o seu gateway de rede virtual e o seu dispositivo VPN no local.

1. No portal Azure, selecione **+Criar um recurso**.
2. Procure **por ligações.**
3. Em **Resultados**, selecione **Connections**.
4. Na **Ligação**, selecione **Criar**.
5. No **Criar Ligação,** configurar as seguintes definições:

    - **Tipo de ligação**: Selecione site-to-site (IPSec).
    - **Grupo de Recursos**: Selecione o seu grupo de recursos de teste.
    - **Virtual Network Gateway**: Selecione a porta de entrada de rede virtual que criou.
    - **Gateway de rede local**: Selecione a porta de entrada de rede local que criou.
    - **Ligação Nome**: Este nome é autopovoado utilizando os valores dos dois gateways.
    - **Tecla partilhada**: Este valor deve corresponder ao valor que está a utilizar para o seu dispositivo VPN local no local. O exemplo tutorial usa 'abc123', mas deve usar algo mais complexo. O importante é que este valor *deve* ser o mesmo valor que especifica ao configurar o seu dispositivo VPN.
    - Os valores de **Subscrição,** **Grupo de Recursos** e **Localização** são fixos.

6. Selecione **OK** para criar a sua ligação.

Pode ver a ligação na página **Connections** do gateway de rede virtual. O estado passará de *Desconhecido* para *Conectado,* e depois para *Sucesso*.

## <a name="next-steps"></a>Passos seguintes

- Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)
