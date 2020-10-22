---
title: Implementar aplicativo híbrido com dados no local que escalam a nuvem cruzada
description: Saiba como implementar uma aplicação que utiliza dados no local e escala em nuvem cruzada usando Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: MT
ms.contentlocale: pt-PT
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353483"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Implementar aplicativo híbrido com dados no local que escalam a nuvem cruzada

Este guia de solução mostra-lhe como implementar uma aplicação híbrida que abrange tanto o Azure como o Azure Stack Hub e utiliza uma única fonte de dados no local.

Ao utilizar uma solução de nuvem híbrida, pode combinar os benefícios de conformidade de uma nuvem privada com a escalabilidade da nuvem pública. Os seus desenvolvedores também podem aproveitar o ecossistema de desenvolvedores da Microsoft e aplicar as suas habilidades nos ambientes de nuvem e no local.

## <a name="overview-and-assumptions"></a>Visão geral e pressupostos

Siga este tutorial para configurar um fluxo de trabalho que permite aos desenvolvedores implementar uma aplicação web idêntica para uma nuvem pública e uma nuvem privada. Esta aplicação pode aceder a uma rede de encaminhamento não internet hospedada na nuvem privada. Estas aplicações web são monitorizadas e quando há um pico no tráfego, um programa modifica os registos DNS para redirecionar o tráfego para a nuvem pública. Quando o tráfego cai para o nível antes do pico, o tráfego é encaminhado de volta para a nuvem privada.

Este tutorial abrange as seguintes tarefas:

> [!div class="checklist"]
> - Implemente um servidor de base de dados SQL Server ligado a híbridos.
> - Ligue uma aplicação web no Azure global a uma rede híbrida.
> - Configure DNS para escalar as nuvens cruzadas.
> - Configure os certificados SSL para a escala de nuvens cruzadas.
> - Configure e implemente a aplicação web.
> - Crie um perfil de Gestor de Tráfego e configuure-o para a escala de nuvens cruzadas.
> - Configurar a monitorização e alerta de Informações de Aplicação para o aumento do tráfego.
> - Configure a troca automática de tráfego entre o Azure global e o Azure Stack Hub.

> [!Tip]  
> ![Diagrama de pilares híbridos](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub é uma extensão do Azure. O Azure Stack Hub traz a agilidade e inovação da computação em nuvem para o seu ambiente no local, permitindo a única nuvem híbrida que lhe permite construir e implementar aplicações híbridas em qualquer lugar.  
> 
> O artigo [Projeto de aplicação híbrido considera](overview-app-design-considerations.md) os pilares da qualidade do software (colocação, escalabilidade, disponibilidade, resiliência, gestão e segurança) para conceber, implementar e operar aplicações híbridas. As considerações de design ajudam na otimização do design de aplicações híbridas, minimizando os desafios em ambientes de produção.

### <a name="assumptions"></a>Pressupostos

Este tutorial pressupõe que você tem um conhecimento básico de Azure global e Azure Stack Hub. Se quiser saber mais antes de iniciar o tutorial, reveja estes artigos:

- [Introdução ao Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Conceitos de chave de hub de pilha de Azure](/azure-stack/operator/azure-stack-overview.md)

Este tutorial também assume que tem uma subscrição do Azure. Se não tiver uma subscrição, [crie uma conta gratuita](https://azure.microsoft.com/free/) antes de começar.

## <a name="prerequisites"></a>Pré-requisitos

Antes de iniciar esta solução, certifique-se de que cumpre os seguintes requisitos:

- Um Kit de Desenvolvimento de Pilhas Azure (ASDK) ou uma subscrição de um Sistema Integrado Azure Stack Hub. Para implementar o ASDK, siga as instruções em [Implementar o ASDK utilizando o instalador](/azure-stack/asdk/asdk-install.md).
- A sua instalação Azure Stack Hub deve ter o seguinte instalado:
  - O Serviço de Aplicações Azure. Trabalhe com o seu Operador Azure Stack Hub para implementar e configurar o Serviço de Aplicações Azure no seu ambiente. Este tutorial requer que o Serviço de Aplicações tenha pelo menos um (1) papel dedicado ao trabalhador disponível.
  - Uma imagem do Windows Server 2016.
  - Um Windows Server 2016 com uma imagem do Microsoft SQL Server.
  - Os planos e ofertas apropriados.
  - Um nome de domínio para a sua aplicação web. Se não tiver um nome de domínio, pode comprar um a um fornecedor de domínio como GoDaddy, Bluehost e InMotion.
- Um certificado SSL para o seu domínio a partir de uma autoridade de certificado de confiança como o LetsEncrypt.
- Uma aplicação web que comunica com uma base de dados do SQL Server e suporta o Application Insights. Você pode baixar a aplicação de amostra [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) do GitHub.
- Uma rede híbrida entre uma rede virtual Azure e a rede virtual Azure Stack Hub. Para obter instruções detalhadas, consulte [a conectividade em nuvem híbrida Configure com O Azure e o Azure Stack Hub](solution-deployment-guide-connectivity.md).

- Um gasoduto híbrido de integração contínua/implantação contínua (CI/CD) com um agente de construção privada no Azure Stack Hub. Para obter instruções detalhadas, consulte [a identidade em nuvem híbrida Configure com aplicações Azure Stack Hub](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Implementar um servidor de base de dados sql ligado híbrido

1. Assine no portal de utilizadores Azure Stack Hub.

2. No **Painel de Instrumentos**, selecione **Marketplace**.

    ![Mercado Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. No **Marketplace**, selecione **Compute**, e depois escolha **Mais**. Em **Mais**, selecione a **Licença de servidor SQL grátis: SQL Server 2017 Developer na** imagem do Servidor do Windows.

    ![Selecione uma imagem de máquina virtual no portal de utilizador Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. Na **Licença de servidor SQL grátis: SQL Server 2017 Developer on Windows Server**, selecione **Create**.

5. Em **Basics > configurações básicas**de configuração , forneça um **Nome** para a máquina virtual (VM), um nome **de utilizador** para o SQL Server SA e uma **Palavra-passe** para a SA.  A partir da lista de drop-down de **subscrição,** selecione a subscrição para a qual está a implementar. Para **o grupo de recursos,** utilize escolha o VM **existente** e coloque o VM no mesmo grupo de recursos que a sua aplicação web Azure Stack Hub.

    ![Configurar definições básicas para VM no portal de utilizador Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. Under **Size,** escolha um tamanho para o seu VM. Para este tutorial, recomendamos A2_Standard ou um DS2_V2_Standard.

7. Em **Definições > configurar funcionalidades opcionais,** configurar as seguintes definições:

   - **Conta de armazenamento**: Crie uma nova conta se precisar de uma.
   - **Rede virtual:**

     > [!Important]  
     > Certifique-se de que o seu SQL Server VM está implantado na mesma rede virtual que os gateways VPN.

   - **Endereço IP público**: Utilize as definições predefinidos.
   - **Grupo de segurança de rede**: (NSG). Criar um novo NSG.
   - **Extensões e Monitorização**: Mantenha as definições predefinidos.
   - **Conta de armazenamento de**diagnóstico: Crie uma nova conta se precisar de uma.
   - Selecione **OK** para guardar a sua configuração.

     ![Configurar funcionalidades opcionais de VM no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. Nas **definições do SQL Server,** configure as seguintes definições:

   - Para **conectividade SQL,** selecione **Public (Internet)**.
   - Para **o Porto**, mantenha o padrão, **1433**.
   - Para **autenticação SQL**, selecione **Ativar**.

     > [!Note]  
     > Quando ativa a autenticação SQL, deve povoar-se automaticamente com a informação "SQLAdmin" que configuraste em **Básicos.**

   - Para o resto das definições, mantenha as predefinições. Selecione **OK**.

     ![Configurar as definições do Servidor SQL no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. Em **Resumo,** reveja a configuração VM e, em seguida, selecione **OK** para iniciar a implementação.

    ![Resumo de configuração no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. Leva algum tempo para criar o novo VM. Pode ver o estado dos seus VMs em **máquinas Virtuais.**

    ![Estado das máquinas virtuais no portal de utilizadores do Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Criar aplicativos web em Azure e Azure Stack Hub

O Azure App Service simplifica a execução e gestão de uma aplicação web. Como o Azure Stack Hub é consistente com o Azure, o Serviço de Aplicações pode funcionar em ambos os ambientes. Utilizará o Serviço de Aplicações para hospedar a sua aplicação.

### <a name="create-web-apps"></a>Criar aplicativos web

1. Crie uma aplicação web em Azure seguindo as instruções em [Gerir um plano de Serviço de Aplicações em Azure.](/azure/app-service/app-service-plan-manage#create-an-app-service-plan) Certifique-se de colocar a aplicação web no mesmo grupo de subscrição e recursos que a sua rede híbrida.

2. Repita o passo anterior (1) no Azure Stack Hub.

### <a name="add-route-for-azure-stack-hub"></a>Adicione rota para Azure Stack Hub

O Serviço de Aplicações no Azure Stack Hub deve ser redirecionável a partir da internet pública para permitir que os utilizadores acedam à sua aplicação. Se o seu Azure Stack Hub estiver acessível a partir da internet, tome nota do endereço IP ou URL virado para o público para a aplicação web Azure Stack Hub.

Se estiver a usar um ASDK, pode [configurar um mapeamento ESTÁTICO NAT](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) para expor o Serviço de Aplicações fora do ambiente virtual.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Conecte uma aplicação web em Azure a uma rede híbrida

Para fornecer conectividade entre a extremidade frontal da web em Azure e a base de dados SQL Server em Azure Stack Hub, a aplicação web deve ser conectada à rede híbrida entre Azure e Azure Stack Hub. Para ativar a conectividade, terá de:

- Configurar a conectividade ponto-a-local.
- Configure a aplicação web.
- Modifique o portal de rede local no Azure Stack Hub.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Configurar a rede virtual Azure para a conectividade ponto-a-local

O gateway de rede virtual no lado Azure da rede híbrida deve permitir que as ligações ponto-a-local se integrem com o Azure App Service.

1. No portal Azure, aceda à página de gateway de rede virtual. Em **Definições**, selecione **configuração ponto-a-local**.

    ![Opção ponto-a-local no gateway de rede virtual Azure](media/solution-deployment-guide-hybrid/image8.png)

2. Selecione **Configurar agora** para configurar ponto a local.

    ![Iniciar configuração ponto-a-local no gateway de rede virtual Azure](media/solution-deployment-guide-hybrid/image9.png)

3. Na página de configuração **ponto-a-local,** insira o intervalo de endereço IP privado que pretende utilizar na **piscina Address**.

   > [!Note]  
   > Certifique-se de que o intervalo especificado não se sobrepõe a nenhuma das gamas de endereços já utilizadas pelas sub-redes nos componentes globais do Azure ou do Azure Stack Hub da rede híbrida.

   Sob **o tipo de túnel,** desmarque a **VPN IKEv2**. **Selecione Guardar** para terminar a configuração ponto a local.

   ![Definições ponto-a-local no gateway de rede virtual Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrar a app Azure App Service com a rede híbrida

1. Para ligar a aplicação ao Azure VNet, siga as instruções em Gateway necessárias para [a integração do VNet](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).

2. Vá a **Definições** para o plano de Serviço de Aplicações que hospeda a aplicação web. Em **Definições**, selecione **Networking**.

    ![Configurar rede para o plano de Serviço de Aplicações](media/solution-deployment-guide-hybrid/image11.png)

3. Na **Integração VNET,** selecione **Clique aqui para gerir.**

    ![Gerir a integração do VNET para o plano de Serviço de Aplicações](media/solution-deployment-guide-hybrid/image12.png)

4. Selecione o VNET que pretende configurar. Nos **endereços IP ROUTED TO VNET,** insira o intervalo de endereços IP para o Azure VNet, o Azure Stack Hub VNet e os espaços de endereço ponto a local. **Selecione Guardar** para validar e guardar estas definições.

    ![Intervalos de endereços IP para rota na Integração de Rede Virtual](media/solution-deployment-guide-hybrid/image13.png)

Para saber mais sobre como o Serviço de Aplicações se integra com os VNets Azure, consulte [Integrar a sua aplicação com uma Rede Virtual Azure.](/azure/app-service/web-sites-integrate-with-vnet)

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Configure a rede virtual Azure Stack Hub

O portal de rede local na rede virtual Azure Stack Hub precisa de ser configurado para encaminhar o tráfego a partir da gama de endereços ponto-a-local do Serviço de Aplicações.

1. No portal Azure Stack Hub, aceda ao **gateway de rede local.** Em **Definições**, selecione **Configuração**.

    ![Opção de configuração gateway em Azure Stack Hub gateway de rede local](media/solution-deployment-guide-hybrid/image14.png)

2. No **espaço Address**, insira a gama de endereços ponto-a-local para o gateway de rede virtual em Azure.

    ![Espaço de endereço ponto-a-local no gateway de rede local Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. **Selecione Guardar** para validar e guardar a configuração.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Configurar DNS para escala de nuvem cruzada

Ao configurar adequadamente o DNS para aplicações cross-cloud, os utilizadores podem aceder às instâncias globais do Azure Stack Hub da sua aplicação web. A configuração do DNS para este tutorial também permite que o Azure Traffic Manager encaminhe o tráfego quando a carga aumenta ou diminui.

Este tutorial utiliza o Azure DNS para gerir o DNS porque os domínios do Serviço de Aplicações não funcionam.

### <a name="create-subdomains"></a>Criar subdomínios

Como o Traffic Manager depende de DNS CNAMEs, é necessário um subdomínio para encaminhar adequadamente o tráfego para os pontos finais. Para obter mais informações sobre registos DNS e mapeamento de domínio, consulte [os domínios do mapa com o Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Para o ponto final do Azure, irá criar um subdomínio que os utilizadores podem usar para aceder à sua aplicação web. Para este tutorial, pode utilizar **app.northwind.com,** mas deve personalizar este valor com base no seu próprio domínio.

Também terás de criar um subdomínio com um disco A para o ponto final do Azure Stack Hub. Pode **usá azurestack.northwind.com.**

### <a name="configure-a-custom-domain-in-azure"></a>Configure um domínio personalizado em Azure

1. Adicione o **nome de anfitrião app.northwind.com** à aplicação web Azure [mapeando um CNAME ao Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Configure domínios personalizados no Azure Stack Hub

1. Adicione o **nome de anfitrião azurestack.northwind.com** à aplicação web Azure Stack Hub [mapeando um disco A ao Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Utilize o endereço IP de encaminhamento de internet para a aplicação Do Serviço de Aplicações.

2. Adicione o **nome de anfitrião app.northwind.com** à aplicação web Azure Stack Hub [mapeando um CNAME para O Serviço de Aplicações Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Utilize o nome de anfitrião configurado no passo anterior (1) como alvo para o CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Configure certificados SSL para escalamento de nuvens cruzadas

É importante garantir que os dados sensíveis recolhidos pela sua aplicação web são seguros em trânsito para e quando armazenados na base de dados SQL.

Irá configurar as suas aplicações web Azure e Azure Stack Hub para utilizar certificados SSL para todo o tráfego de entrada.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Adicione SSL ao Azure e ao Azure Stack Hub

Para adicionar SSL a Azure:

1. Certifique-se de que o certificado SSL que obtém é válido para o subdomínio que criou. (Não faz mal usar certificados wildcard.)

2. No portal Azure, siga as instruções na **aplicação Web** e ligue as secções de **certificado SSL** do [Bind um certificado SSL personalizado existente ao artigo da Azure Web Apps.](/azure/app-service/app-service-web-tutorial-custom-ssl) Selecione **SSL baseado em SNI** como o **Tipo SSL**.

3. Redirecione todo o tráfego para a porta HTTPS. Siga as instruções na secção   **HttpS da Secção HTTPS** do [Vinco um certificado SSL personalizado existente para](/azure/app-service/app-service-web-tutorial-custom-ssl) o artigo da Azure Web Apps.

Para adicionar SSL ao Azure Stack Hub:

1. Repita os passos 1-3 que usou para o Azure, utilizando o portal Azure Stack Hub.

## <a name="configure-and-deploy-the-web-app"></a>Configurar e implementar a aplicação Web

Irá configurar o código da aplicação para reportar telemetria à instância correta do Application Insights e configurar as aplicações web com as cadeias de conexão certas. Para saber mais sobre a Aplicação Insights, veja [o que é Insights de Aplicação?](/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Adicionar Insights de Aplicação

1. Abra a sua aplicação web no Microsoft Visual Studio.

2. [Adicione Insights de Aplicação](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) ao seu projeto para transmitir a telemetria que o Application Insights utiliza para criar alertas quando o tráfego web aumenta ou diminui.

### <a name="configure-dynamic-connection-strings"></a>Configurar cordas de ligação dinâmica

Cada instância da aplicação web usará um método diferente para ligar à base de dados SQL. A aplicação em Azure utiliza o endereço IP privado do SQL Server VM e a aplicação no Azure Stack Hub utiliza o endereço IP público do SQL Server VM.

> [!Note]  
> Num sistema integrado Azure Stack Hub, o endereço IP público não deve ser dirijo. Numa ASDK, o endereço IP público não é roteado fora da ASDK.

Pode utilizar variáveis ambientais do Serviço de Aplicações para passar uma cadeia de ligação diferente a cada instância da aplicação.

1. Abra a aplicação no Visual Studio.

2. Abra Startup.cs e encontre o seguinte bloco de código:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Substitua o bloco de código anterior pelo seguinte código, que utiliza uma cadeia de ligação definida no *appsettings.jsno* ficheiro:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Configurar configurar as configurações da aplicação do Serviço de Aplicações de Aplicações

1. Crie cordas de conexão para Azure e Azure Stack Hub. As cordas devem ser as mesmas, exceto os endereços IP que são utilizados.

2. No Azure e no Azure Stack Hub, adicione a cadeia de conexão apropriada [como uma definição](/azure/app-service/web-sites-configure) de aplicação na aplicação web, usando `SQLCONNSTR\_` como um prefixo no nome.

3. **Guarde** as definições da aplicação web e reinicie a aplicação.

## <a name="enable-automatic-scaling-in-global-azure"></a>Permitir a escala automática no Azure global

Quando cria a sua aplicação web num ambiente de Serviço de Aplicações, começa com um exemplo. Pode escalar automaticamente para adicionar instâncias para fornecer mais recursos computacional para a sua aplicação. Da mesma forma, pode escalar automaticamente e reduzir o número de casos de que a sua aplicação necessita.

> [!Note]  
> Precisa de ter um plano de Serviço de Aplicações para configurar escala e escalar. Se não tem um plano, crie um antes de começar os próximos passos.

### <a name="enable-automatic-scale-out"></a>Permitir a escala automática

1. No portal Azure, encontre o plano de Serviço de Aplicações para os sites que pretende escalar e, em seguida, selecione **Scale-out (Plano de Serviço de Aplicações)**.

    ![Dimensione o Serviço de Aplicações Azure](media/solution-deployment-guide-hybrid/image16.png)

2. **Selecione Ative autoscale**.

    ![Permitir a autoescalação no Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. Introduza um nome para **Nome de Definição de Escala Automática**. Para a regra de escala automática **predefinido,** selecione **Escala com base numa métrica**. Definir os **limites de instância** para **mínimo: 1,** **Máximo: 10**, e **Predefinido: 1**.

    ![Configurar autoescalação no Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. **Selecione +Adicionar uma regra**.

5. Na **Fonte Métrica**, selecione **O Recurso Atual**. Utilize os seguintes critérios e ações para a regra.

#### <a name="criteria"></a>Critérios

1. Em **Agregação de Tempo,** selecione **Média**.

2. Em **Nome Métrico**, selecione **PERCENTAGEM CPU**.

3. No **Operador**, selecione **Maior que**.

   - Definir o **Limiar** para **50**.
   - Definir a **Duração** para **10**.

#### <a name="action"></a>Ação

1. Em **operação**, selecione **Aumentar a Contagem por**.

2. Definir o **Conde de Instância** para **2**.

3. **Desafrie** o arrefecimento para **5**.

4. Selecione **Adicionar**.

5. Selecione o **+ Adicionar uma regra**.

6. Na **Fonte Métrica,** selecione **O Recurso Atual.**

   > [!Note]  
   > O recurso atual conterá o nome/GUID do seu plano de Serviço de Aplicações e as listas de drop-down tipo de **recurso** e **recursos** não estarão disponíveis.

### <a name="enable-automatic-scale-in"></a>Ativar a escala automática em

Quando o tráfego diminui, a aplicação web Azure pode reduzir automaticamente o número de casos ativos para reduzir custos. Esta ação é menos agressiva do que a escala e minimiza o impacto nos utilizadores de aplicações.

1. Vá para a condição de escala **padrão** e, em seguida, **selecione + Adicione uma regra**. Utilize os seguintes critérios e ações para a regra.

#### <a name="criteria"></a>Critérios

1. Em **Agregação de Tempo,** selecione **Média**.

2. Em **Nome Métrico**, selecione **PERCENTAGEM CPU**.

3. No **Operador**, selecione **Menos de**.

   - Definir o **Limiar** para **30**.
   - Definir a **Duração** para **10**.

#### <a name="action"></a>Ação

1. Em **operação**, **selecione Decrease Count by**.

   - Decrete o **Conde de Instância** para **1**.
   - **Desafrie** o arrefecimento para **5**.

2. Selecione **Adicionar**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Crie um perfil de Gestor de Tráfego e configuure a escala de nuvens cruzadas

Crie um perfil de Gestor de Tráfego utilizando o portal Azure e, em seguida, configuure pontos finais para permitir a escala de nuvens cruzadas.

### <a name="create-traffic-manager-profile"></a>Criar perfil de Gestor de Tráfego

1. Selecione **Criar um recurso**.
2. Selecione **Rede**.
3. Selecione **o perfil do Gestor de Tráfego** e configufique as seguintes definições:

   - Em **Nome,** insira um nome para o seu perfil. Este nome **deve** ser único na zona trafficmanager.net e é usado para criar um novo nome DNS (por exemplo, northwindstore.trafficmanager.net).
   - Para **o método de encaminhamento,** selecione o **Ponderado**.
   - Para **Subscrição**, selecione a subscrição em que pretende criar este perfil.
   - No **Grupo de Recursos,** crie um novo grupo de recursos para este perfil.
   - Em **Localização do grupo de recursos**, selecione a localização do grupo de recursos. Esta definição refere-se à localização do grupo de recursos e não tem qualquer impacto no perfil do Gestor de Tráfego que é implantado globalmente.

4. Selecione **Criar**.

    ![Criar perfil de Gestor de Tráfego](media/solution-deployment-guide-hybrid/image19.png)

   Quando a implementação global do seu perfil de Gestor de Tráfego está completa, é mostrada na lista de recursos para o grupo de recursos que o criou.

### <a name="add-traffic-manager-endpoints"></a>Adicionar pontos finais do Gestor de Tráfego

1. Procure o perfil de Gestor de Tráfego que criou. Se navegou para o grupo de recursos para o perfil, selecione o perfil.

2. No **perfil de Gestor de Tráfego**, em **DEFINIÇÕES,** selecione **Pontos de final**.

3. Selecione **Adicionar**.

4. No **ponto final adicionar,** utilize as seguintes definições para Azure Stack Hub:

   - Para **o tipo**, selecione ponto final **externo**.
   - Insira um **nome** para o ponto final.
   - Para **nome de domínio totalmente qualificado (FQDN) ou IP,** insira o URL externo para a sua aplicação web Azure Stack Hub.
   - Para **o peso**, mantenha o padrão, **1**. Este peso resulta em todo o tráfego indo para este ponto final se for saudável.
   - Deixe **adicionar como desativado** sem verificação.

5. Selecione **OK** para guardar o ponto final do Azure Stack Hub.

Vai configurar o ponto final do Azure a seguir.

1. No **perfil de Gestor de Tráfego**, selecione **Pontos de final**.
2. Selecione **+Adicionar**.
3. No **ponto final adicionar,** utilize as seguintes definições para Azure:

   - Para **tipo**, selecione **Azure endpoint**.
   - Insira um **nome** para o ponto final.
   - Para **o tipo de recurso Alvo**, selecione App **Service**.
   - Para **o recurso Target**, **selecione Escolha um serviço** de aplicações para ver uma lista de Aplicações Web na mesma subscrição.
   - Em **Recurso**, escolha o Serviço de Aplicações que pretende adicionar como o primeiro ponto final.
   - Para **peso**, selecione **2**. Esta definição resulta em todo o tráfego que vai para este ponto final se o ponto final principal não for saudável, ou se tiver uma regra/alerta que redirecione o tráfego quando ativado.
   - Deixe **adicionar como desativado** sem verificação.

4. Selecione **OK** para guardar o ponto final do Azure.

Depois de configurados ambos os pontos finais, estão listados no **perfil de Gestor de Tráfego** quando seleciona **Pontos de Final**. O exemplo na seguinte captura de ecrã mostra dois pontos finais, com informações de estado e de configuração para cada um.

![Pontos finais no perfil do Gestor de Tráfego](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a>Configurar a monitorização e alerta de Insights de Aplicação em Azure

O Azure Application Insights permite-lhe monitorizar a sua aplicação e enviar alertas com base nas condições que configura. Alguns exemplos são: a aplicação está indisponível, está a sofrer falhas ou está a apresentar problemas de desempenho.

Utilizará as métricas Azure Application Insights para criar alertas. Quando estes alertas disparam, a instância da sua aplicação web mudará automaticamente de Azure Stack Hub para Azure para escalar e, em seguida, de volta para Azure Stack Hub para escalar dentro

### <a name="create-an-alert-from-metrics"></a>Criar um alerta a partir de métricas

No portal Azure, vá ao grupo de recursos para este tutorial e selecione a instância Application Insights para abrir **Insights de Aplicação**.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Você usará esta vista para criar um alerta de escala e um alerta de escala.

### <a name="create-the-scale-out-alert"></a>Crie o alerta de escala

1. Em **CONFIGURE,** **selecione Alertas (clássicos)**.
2. **Selecione Adicionar alerta métrico (clássico)**.
3. Na **regra adicionar,** configurar as seguintes definições:

   - Para **nome**, **insira Burst into Azure Cloud**.
   - Uma **descrição** é opcional.
   - Em Alerta **de Fonte**  >  **em**, selecione **Métricas**.
   - De **Acordo com critérios,** selecione a sua subscrição, o grupo de recursos para o seu perfil de Gestor de Tráfego e o nome do perfil de Gestor de Tráfego para o recurso.

4. Para **métrica**, selecione **Taxa de Pedido**.
5. Para **a condição**, selecione Maior **que**.
6. Para **limiar**, insira **2**.
7. Para **o período**, selecione Ao longo dos **últimos 5 minutos**.
8. Em **notificação através de:**
   - Verifique a caixa de verificação para **os proprietários, colaboradores e leitores de e-mail.**
   - Insira o seu endereço de e-mail para **e-mails adicionais do administrador.**

9. Na barra de menus, **selecione Save**.

### <a name="create-the-scale-in-alert"></a>Crie o alerta de escala

1. Em **CONFIGURE,** **selecione Alertas (clássicos)**.
2. **Selecione Adicionar alerta métrico (clássico)**.
3. Na **regra adicionar,** configurar as seguintes definições:

   - Para **nome**, **introduza a Escala de volta no Azure Stack Hub**.
   - Uma **descrição** é opcional.
   - Em Alerta **de Fonte**  >  **em**, selecione **Métricas**.
   - De **Acordo com critérios,** selecione a sua subscrição, o grupo de recursos para o seu perfil de Gestor de Tráfego e o nome do perfil de Gestor de Tráfego para o recurso.

4. Para **métrica**, selecione **Taxa de Pedido**.
5. Para **a condição**, selecione Menos **de**.
6. Para **limiar**, insira **2**.
7. Para **o período**, selecione Ao longo dos **últimos 5 minutos**.
8. Em **notificação através de:**
   - Verifique a caixa de verificação para **os proprietários, colaboradores e leitores de e-mail.**
   - Insira o seu endereço de e-mail para **e-mails adicionais do administrador.**

9. Na barra de menus, **selecione Save**.

A imagem que se segue mostra os alertas para a escala e a escala.

   ![Alertas de Insights de Aplicação (clássico)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Redirecione o tráfego entre Azure e Azure Stack Hub

Pode configurar a comutação manual ou automática do tráfego da sua aplicação web entre o Azure e o Azure Stack Hub.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Configurar a troca manual entre O Azure e o Azure Stack Hub

Quando o seu site atingir os limiares que configura, receberá um alerta. Utilize os seguintes passos para redirecionar manualmente o tráfego para Azure.

1. No portal Azure, selecione o seu perfil de Gestor de Tráfego.

    ![Pontos finais do Gestor de Tráfego no portal Azure](media/solution-deployment-guide-hybrid/image20.png)

2. Selecione **Pontos de final**.
3. Selecione o **ponto final Azure**.
4. Em **Status**, selecione **Ativado**e, em seguida, selecione **Guardar**.

    ![Ativar o ponto final do Azure no portal Azure](media/solution-deployment-guide-hybrid/image23.png)

5. Em **Pontos de fim** para o perfil de Gestor de Tráfego, selecione Ponto final **Externo**.
6. Em **Estado de Estado**, selecione **Desativado**e, em seguida, selecione **Guardar**.

    ![Desativar o ponto final do Azure Stack Hub no portal Azure](media/solution-deployment-guide-hybrid/image24.png)

Depois de configurados os pontos finais, o tráfego de aplicações vai para a sua aplicação web de escala Azure em vez da aplicação web Azure Stack Hub.

 ![Pontos finais alterados no tráfego de aplicações web Azure](media/solution-deployment-guide-hybrid/image25.png)

Para inverter o fluxo de volta para Azure Stack Hub, use os passos anteriores para:

- Ativar o ponto de terminação do Azure Stack Hub.
- Desative o ponto final do Azure.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Configurar a comutação automática entre O Azure e o Azure Stack Hub

Também pode utilizar a monitorização do Application Insights se a sua aplicação funcionar num ambiente [sem servidor](https://azure.microsoft.com/overview/serverless-computing/) fornecido pelas Funções Azure.

Neste cenário, pode configurar o Application Insights para usar um webhook que chama uma aplicação de função. Esta aplicação ativa ou desativa automaticamente um ponto final em resposta a um alerta.

Utilize os seguintes passos como guia para configurar a comutação automática de tráfego.

1. Crie uma aplicação Azure Function.
2. Crie uma função acionada por HTTP.
3. Importe os Azure SDKs para Gestor de Recursos, Web Apps e Traffic Manager.
4. Desenvolver código para:

   - Autenticar a sua assinatura Azure.
   - Utilize um parâmetro que alterne os pontos finais do Gestor de Tráfego para direcionar o tráfego para o Azure ou para o Azure Stack Hub.

5. Guarde o seu código e adicione o URL da aplicação de função com os parâmetros apropriados à secção **Webhook** das definições de regra de alerta de Insights de Aplicação.
6. O tráfego é redirecionado automaticamente quando um alerta de Insights de Aplicação dispara.

## <a name="next-steps"></a>Passos seguintes

- Para saber mais sobre padrões de nuvem azure, consulte [padrões de design de nuvem.](/azure/architecture/patterns)
