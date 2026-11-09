# Diagnóstico do Ambiente — DevOps

**Disciplina:** DevOps — Unidade 3  
**Integrantes:** Lucas José Monteiro Sales Barreto e Caio Gama da Silva Campos

## Problema 1 — Bancos de dados em versões diferentes

**Problema identificado:** Os ambientes utilizam versões diferentes do banco de dados, o que pode provocar incompatibilidades entre desenvolvimento, testes e produção.

**Consequência:** Uma aplicação pode funcionar corretamente em um ambiente e apresentar erros em outro por causa das diferenças entre as versões.

**Camada afetada:** Ambiente e infraestrutura.

**Classificação da intervenção:** Configuração e provisionamento.

**Estado desejado:** Todos os ambientes devem utilizar versões padronizadas e documentadas do banco de dados.

**Como comprovar a correção:** Consultar a versão do banco em cada ambiente e verificar se todas correspondem à versão definida no projeto.

## Problema 2 — Porta liberada manualmente

**Problema identificado:** Uma porta de comunicação foi configurada manualmente, sem registro adequado no código do projeto.

**Consequência:** Caso o ambiente precise ser recriado, a configuração pode ser esquecida, impedindo o acesso à aplicação.

**Camada afetada:** Rede e configuração do ambiente.

**Classificação da intervenção:** Configuração.

**Estado desejado:** A configuração da porta deve estar documentada e definida em arquivos de configuração versionados.

**Como comprovar a correção:** Recriar o ambiente utilizando a configuração do projeto e verificar se a aplicação continua acessível pela porta esperada.

## Problema 3 — Dependência de uma única pessoa

*A ser desenvolvido pelo Integrante 2.*

## Conclusão

*A ser finalizada pelos dois integrantes após a análise dos três problemas.*