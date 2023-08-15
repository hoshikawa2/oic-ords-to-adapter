- Estratégia para justificar ORDS vs Adapter
    - Integração principal: Mantém
    - Criar 2 integrações específicas de interface com o banco
        - ORDS: JSON mesmo
        - Adapter: JSON mesmo
        - Mudanças ocorrem na reescrita deste componente
        - Excluir o artefato ORDS e Importar o artefato Adapter: OK?????
- Estratégia para Performance: Tempos aqui são diferentes. Queries maiores tendem a ter o mesmo tempo em ORDS ou Adapter (TESTAR HIPOTESE)
    - Integração Principal
    - Integração de chamada ao banco
        - Warm-up de conector
            - Adapter
            - ORDS
        - Transmissão da query para Banco de Dados
        - Processamento de Banco de Dados
        - Retorno dos dados
            - Adapter
            - ORDS

# Integrações com Banco de Dados para Oracle Integration

### Pre-Requisitos

- ORDS devidamente configurado ao banco de dados alvo
- Tabelas e queries devidamente configuradas para consumo através de serviço REST
- OIC devidamente configurado para acessar os serviços REST do ORDS

### Oracle REST Data Source (ORDS)

Oracle REST Data Services (ORDS) faz a ponte entre o HTTPS e o seu Oracle Database. Um aplicativo Java de camada intermediária, o ORDS fornece uma API REST de Gerenciamento de Banco de Dados, o SQL Developer Web, um Gateway PL/SQL, SODA para REST e capacidade de publicar Serviços da Web em RESTful para interagir com os dados e procedimentos armazenados no Oracle Database.

### Vantagens de utilizar ORDS

- Isolamento do Banco de Dados para acessos diretos
- Camada REST API 
- Evita necessidade de importar drivers de banco de dados na aplicação
- Boa performance

### Migrando ORDS para OIC Database Adapter

Para tornar a tarefa de migrar ORDS para o Adapter de Banco de Dados de OIC com o menor esforço possível, devemos manter em mente:

- Implementar a integração principal com chamadas para integrações específicas com acesso a banco. Em outras palavras, implementar integrações pontuais específicas de acesso ao banco de dados e chamá-las na implementação da integração principal. Assim, quando houver necessidade de migração, basta trocar a implementação da integração de banco e ajustar o apontamento na integração principal.
- Na integração com adapter, implementar como trigger para REST
- Criar as integrações com banco ORDS e Adapter com o mesmo nome 
- Criar as integrações com banco com uma mesma interface de entrada e de saída, com as mesmas estruturas JSON
- Criar as integrações com banco com o mesmo path REST

Ao chamar a Integração específica de banco de dados, implementar a chamada com o Local Integration.
Ao trocar o apontamento de ORDS para Adapter, não haverá perda de mapeamentos

![img_1.png](img_1.png)

### Criar a integração de chamada ao banco de dados por ORDS

Criar uma conexão para o ORDS através de um Adapter REST conforme abaixo. Importante definir a URL endpoint para o serviço ORDS e suas credenciais de acesso.

![img_3.png](img_3.png)

Como padrão, criar uma integração como trigger REST e estabelecer um caminho de chamada como mostra o exemplo abaixo:

![img_4.png](img_4.png)

![img_5.png](img_5.png)

![img_6.png](img_6.png)

![img_10.png](img_10.png)

![img_11.png](img_11.png)

![img_12.png](img_12.png)

![img_13.png](img_13.png)

![img_14.png](img_14.png)

> Importante: Esta parte da configuração acima deve ser idêntica tanto para definir acessos do tipo ORDS como do tipo OIC Adapter.

Agora vamos configurar a chamada ao banco de dados através do serviço ORDS:

![img_15.png](img_15.png)

![img_16.png](img_16.png)

![img_17.png](img_17.png)

![img_18.png](img_18.png)

Você vai precisar chamar o serviço ORDS correspondente para obter o JSON e incluir na configuração do OIC:

![img_19.png](img_19.png)

Agora inclua o JSON obtido no OIC e finalize a configuração:

![img_20.png](img_20.png)

Pronto! Agora vamos finalizar as cofigurações mapeando os parâmetros de entrada e saída do serviço
![img_21.png](img_21.png)

![img_22.png](img_22.png)

![img_23.png](img_23.png)

Clique em Validate e Close para finalizar a configuração de entrada

![img_24.png](img_24.png)

Agora vamos configurar o retorno

![img_25.png](img_25.png)

![img_26.png](img_26.png)

Clique em Validate e Close e finalize a configuração

Finalize a configuração se sua chamada REST em ORDS com a parametrização dos dados de tracking, Salve a integração e ative-a:

![img_27.png](img_27.png)

![img_28.png](img_28.png)




> Importante: Defina as estruturas JSON de entrada e saída mais o caminho do serviço REST de forma idêntica para implementações com chamadas ORDS e Adapter OIC

### Criar a integração de chamada ao banco de dados por Adapter

Criar uma conexão para o Banco de Dados Oracle através de seu Adapter correspondente conforme abaixo. Importante definir as credenciais de acesso ao banco de dados (Wallet, usuário/senha ou outro método adequado)

![img_2.png](img_2.png)

Como padrão, criar uma integração como trigger REST e estabelecer um caminho de chamada como mostra o exemplo abaixo:

Defina também a estrutura JSON para os parâmetros de entrada e também os de saída.

Defina como será feita a query

Defina o mapeamento da query para o JSON de saída

### Criar a integração principal com chamada para o banco de dados

A integração principal sempre deve representar a orquestração e os processos principais, deixando que as chamadas de banco de dados sejam feitas por outra integrações em separado.
Isto dará a capacidade de migração sem muito esforço no caso de necessidade de mudar o método de acesso (ORDS para Adapter).

Desta forma, definir a orquestração e realizar as chamadas ao banco de dados através de uma chamada do tipo "Local" conforme abaixo:

Realizar o mapeamento da entrada e saída conforme a definição da estrutura JSON

### Promovendo a troca de ORDS para Adapter

Promover a troca das chamadas ORDS para Adapter é bem simples. Se a estrutura de entrada e saída e também o caminho da chamada REST são idênticas, basta:

- Deletar as integrações pontuais de banco de dados e substituir pelas ORDS

Ao deletar o artefato de acesso ao banco via ORDS, os mapeamentos continuarão presentes na integração principal. Basta então importar o artefato novo equivalente à chamada ao banco (artefato com chamada ao banco de dados via adapter OIC). Ao importar esse novo artefato, o OIC manterá todos os mapeamentos anteriores, preservando a chamada sem necessidade de maiores configurações.

### Referências

* [Build Your First REST Integration](https://apexapps.oracle.com/pls/apex/f?p=44785:50:106332613417553:::50:P50_COURSE_ID,P50_EVENT_ID:344,6079)
* [Oracle REST Data Services (ORDS) best practices](https://www.oracle.com/database/technologies/appdev/rest/best-practices/)
* [Improving the performance of Oracle Integration flows that use REST calls](https://www.ateam-oracle.com/post/improving-the-performance-of-oracle-integration-flows-that-use-rest-calls)
* [Armazenando Respostas no Cache para Melhorar o Desempenho](https://docs.oracle.com/pt-br/iaas/Content/APIGateway/Tasks/apigatewayresponsecaching.htm)
* [Using the Oracle Database Adapter with Oracle Integration Generation 2](https://docs.oracle.com/en/cloud/paas/integration-cloud/database-adapter/index.html#Oracle®-Cloud)

### Estratégia para implementar integrações 

    - Integração principal: Mantém
    - Criar 2 integrações específicas de interface com o banco
        - ORDS: JSON mesmo
        - Adapter: JSON mesmo
        - Mudanças ocorrem na reescrita deste componente
        - Excluir o artefato ORDS e Importar o artefato Adapter: OK?????


