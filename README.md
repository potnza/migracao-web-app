# Projeto final - AWS Faculdade SENAC

**CASE:**

Nós somos da empresa "Fast Engineering S/A"
e gostaríamos de uma solução dos senhores(as),
que fazem parte da empresa terceira "TI
SOLUÇÕES INCRÍVEIS".

Nosso eCommerce está crescendo e a solução
atual não está atendendo mais a alta demanda de
acessos e compras que estamos tendo.

Desde o Início do ano os acessos e compras
estão crescendo 20% a cada mês.
Atualmente usamos:
- 01 servidor para Banco de Dados Mysql;
- 01 servidor para a aplicação utilizando REACT;
- 01 servidor de web Server e que armazena
estáticos como fotos e links.

**Nosso pedido é um *ORÇAMENTO* com:**

- ESCOPO;

- ARQUITETURA DA NOVA SOLUÇÃO;

- VALORES;

- PRAZO DE ENTREGA;

- CRONOGRAMA MACRO DE ENTREGAS;

Sobre a construção da arquitetura para o futuro
website da nossa empresa, precisamos seguir as
melhores práticas DevOps.

**O que é requerido na atividade:**

Ambiente Kubernetes;

Banco de dados PaaS;

MultiAZ;

Segurança de backup de dados;
Persistência dos dados;

Balanceamento de carga com healthcheck;

Segurança (liberar somente o
necessário/mínimo acesso possível).

**Objetivo:** Monte a proposta e a arquitetura do que
o grupo propõe entregar.

## Descrição geral da arquitetura: 
O acesso à aplicação é configurado no domínio do site no **Rota 53**, o serviço de DNS da AWS. O tráfego de entrada passa por um **WAF – Web Application Firewall**, que é uma camada a mais de segurança para o ambiente. O tráfego é direcionado para um **ALB – Application Load Balancer**, que o distribui entre as máquinas do cluster Kubernetes. O cluster Kubernetes é implementado com o serviço de **EKS – Elastic Kubernetes Service**, o qual gerencia o cluster e contém um **AutoScaling **para as máquinas. As máquinas do cluster rodam a aplicação. Elas se conectam ao banco de dados da aplicação, que é uma instância Amazon RDS Aurora. Os snapshots de backup do RDS são armazenados num **Bucket S3 Glacier Instant Retrieval**. Para comunicação externa, as máquinas do cluster contam com o **NAT Gateway**, o elemento que possibilita conexão externa para elas. O ambiente ainda oferece um **EFS – Elastic File System**, o serviço de NFS da Amazon, que é montado nas máquinas armazenando arquivos compartilhados da aplicação. Os estáticos do site são armazenados num Bucket S3 Standard.

####Camada de rede/entrada:

A resolução de DNS do site é feita com o **Rota 53**;
O tráfego é filtrado pelo **WAF (Web Application Firewall)**;
O **ALB (Application Load Balancer) **distribui o tráfego entre os nodes do cluster Kubernetes;

####Camada de aplicação:
O cluster Kubernetes é implementado com o **EKS (Elastic Kubernetes Service)**, que gerencia autonomamente os nodes;
Os nodes do cluster são instâncias EC2 do tipo **M6g.medium**;
O cluster **EKS** contém um **AutoScaling**, serviço que garante dimensionamento horizontal flexível. Inicialmente, é definida a quantidade de 4 máquinas (2 em cada AZ);
Camada de dados:
O **Amazon RDS Aurora** é o banco de dados da aplicação. O Amazon Aurora ja é desenhado para ser multi-az, funcionando por um cluster de data-bases, tendo assim um grande poder de *Disaster Recovery* ou seja, possui uma confiabilidade muito alta.
O **Amazon S3** é usado para armazenamento de estáticos do site e de snapshots de backup do RDS;
Um **EFS (Elastic File System)** é utilizado para armazenamento de arquivos compartilhados da aplicação;

####Acesso administrativo:
O ambiente interno da aplicação pode ser acessado através de um bastion host, que é uma instância EC2, tipo **t2.micro**. Este bastion pode ser usado para fins de acesso ao banco de dados, aos arquivos compartilhados (EFS), a ferramentas de monitoramento… Existe um** AutoScaling Group **configurado para a instância Bastion, definido para as duas Azs, garantindo alta disponibilidade de acesso administrativo.
São definidas credenciais de acesso temporário, roles (funções de permissionamento) e demais mecanismos de segurança com os serviços de **IAM (Identity Access Management)**;

####Configuração de rede:
A aplicação é hospedada na região AWS "sa-east-1" (São Paulo) e dividida entre duas Zonas de Disponibilidade, sa-east-1a e sa-east-1b, garantindo alta disponibilidade.
A rede interna da aplicação é implementada com uma **VPC (Virtual Private Network)**, abrangendo as duas AZs (sa-east-1a e sa-east-1b);
Dentro da VPC existe: uma **subnet pública**, com rotas para um **Internet Gateway.** Ela contém o Bastion Host, o **Nat gateway** e** Load Balancer**; uma subnet privada, contendo os nodes do** cluster EKS**; uma** subnet privada**, contendo o banco de dados ** RDS Aurora**.


