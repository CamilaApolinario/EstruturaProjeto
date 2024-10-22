# EstruturaProjeto

1. Camadas de Arquitetura:
1.1 Camada de Front-End (UI e UX)
Aplicação Web/Mobile: Interface amigável e intuitiva para o usuário realizar login, visualizar ingressos disponíveis e efetuar a compra.
Serviços de Cache para Carga Estática: Implementar CDN para entregar rapidamente conteúdo estático (imagens, scripts, etc.), diminuindo a carga dos servidores web.
Fila de Espera Virtual: Ao acessar o site, os usuários são colocados em uma fila de espera (controlada por um serviço de fila como Redis Queue ou AWS SQS). Isso garante que todos tenham uma chance justa de entrar no sistema de vendas, independentemente da velocidade de conexão.
1.2 Camada de Backend (Serviços de Aplicação)
Serviço de Login (Autenticação e Autorização): Um serviço centralizado de login/autenticação para garantir que apenas usuários autenticados participem da compra de ingressos, com suporte a autenticação por OAuth2, etc.
Serviço de Controle de Sessões: Após o login, gerenciar o tempo de sessão do usuário, liberando a sessão após expiração ou término da compra.
Serviço de Catálogo de Ingressos: Esse serviço controla a exibição de ingressos disponíveis, sempre consultando em tempo real o estoque atual para evitar exibir ingressos já esgotados.
Serviço de Reserva de Ingressos: Quando o usuário seleciona um ingresso, ele não é imediatamente vendido, mas sim reservado temporariamente para o cliente por um tempo limitado (por exemplo, 5 minutos). Se o pagamento não for completado, o ingresso é liberado para outros compradores. Esse serviço precisa ser thread-safe para garantir que dois clientes não reservem o mesmo ingresso.


1.3 Camada de Banco de Dados (Persistência)
Banco de Dados Relacional (SQL): Gerencia as informações de estoque de ingressos, clientes, compras e transações de pagamento. Utilizamos transações para garantir que a atualização do estoque seja atômica e que não ocorram reservas ou vendas de ingressos que já tenham sido vendidos.
Redis Cache para Ingressos Disponíveis: Um sistema de cache in-memory (como Redis) é usado para armazenar ingressos disponíveis em tempo real. Cada vez que um ingresso é reservado ou vendido, ele é removido do cache. Isso reduz a carga no banco de dados e melhora a performance em cenários de alta concorrência.
1.4 Camada de Orquestração e Controle de Concorrência
Serviço de Controle de Concorrência (Locking System): Esse serviço usa bloqueios distribuídos (distributed locks, como no Redis ou ZooKeeper) para garantir que o mesmo ingresso não possa ser reservado ou vendido por dois usuários simultaneamente.
Sistema de Fila de Reservas: Para evitar que todos os usuários acessem o sistema de compra ao mesmo tempo, uma fila de compras (por exemplo, utilizando RabbitMQ ou AWS SQS) é usada para processar as requisições de compra em ordem. Assim, cada requisição de compra é tratada de forma sequencial, sem sobrecarga no sistema de pagamentos.
1.5 Camada de Pagamentos
Gateway de Pagamentos: Quando o usuário confirma a reserva, ele é redirecionado para o pagamento. Integração com diversos gateways de pagamento (cartão de crédito, PayPal, etc.). Após a confirmação do pagamento, o ingresso é marcado como "vendido" no banco de dados.
1.6 Monitoramento e Escalabilidade
Auto-Scaling: Utilizar um serviço de auto-scaling para lidar com picos de demanda, garantindo que novos servidores de aplicação sejam automaticamente iniciados quando houver necessidade (em plataformas como AWS EC2, Kubernetes ou Azure).
Monitoramento em Tempo Real: Utilizar ferramentas como Prometheus e Grafana para monitorar a saúde do sistema (disponibilidade, latência, falhas de pagamento) e gerar alertas automáticos em caso de problemas.

2. Fluxo da Venda de Ingressos:
O usuário entra no site e é colocado em uma fila de espera virtual.
Ao ser direcionado para a página de compra, o usuário efetua o login.
O catálogo de ingressos é exibido a partir de dados em cache para evitar múltiplos acessos simultâneos ao banco de dados.
O usuário seleciona um ingresso e o reserva temporariamente (gerando um lock no sistema para garantir que esse ingresso não seja selecionado por outro usuário).
O ingresso é mantido reservado por um tempo limitado (por exemplo, 5 minutos), e o usuário é redirecionado ao pagamento.
Caso o pagamento seja concluído, o sistema marca o ingresso como vendido.
Se o pagamento falhar ou o tempo de reserva expirar, o ingresso é liberado novamente para o próximo comprador.



