# DevOps: gerenciamento e provisionamento automatizado de ambientes

**Prof. Dr. Felipe Cordeiro — Unidade 3**

Este kit acompanha a aula e contém três experiências: publicar uma página com Docker, administrar um arquivo com Terraform e manter uma configuração com Ansible. O laboratório usa o navegador para acessar um ambiente Linux remoto no GitHub Codespaces.

## 1. Preparação do repositório e do Codespace

### 1.1. O que precisa estar disponível

- Uma conta GitHub com acesso ao Codespaces e conexão com a internet.
- O pacote da atividade extraído no computador, ou um repositório da turma já preparado pelo professor.
- Tempo para a primeira preparação do ambiente, que baixa ferramentas e dependências.

O computador da sala não precisa receber instalações de Docker, Terraform, Ansible ou VS Code. Essas ferramentas serão preparadas no ambiente remoto.

### 1.2. Enviar o conteúdo do kit

1. No GitHub, crie um repositório pessoal chamado `devops-ambientes-aula`, com um `README.md` inicial. Para a atividade em dupla, use o repositório combinado com o professor e adicione o colega como colaborador.
2. Abra **Add file → Upload files** e arraste o **conteúdo** da pasta extraída. A pasta `.devcontainer` precisa ficar diretamente na raiz do repositório, junto de `terraform`, `ansible` e `docker`.
3. Confira se a lista de envio inclui `.devcontainer/devcontainer.json`, `.devcontainer/setup.sh` e `.devcontainer/requirements.txt`. Não envie uma pasta externa adicional que deixe esses arquivos em um nível abaixo.
4. Confirme o commit inicial. Depois, confira os arquivos na página do repositório antes de abrir o Codespace.

Se a pasta `.devcontainer` não aparecer no seletor de arquivos, habilite a exibição de itens ocultos no computador. Também é possível usar **Add file → Create new file**, informar o nome completo, como `.devcontainer/devcontainer.json`, e colar o conteúdo correspondente do kit. A barra no nome cria a pasta.

| Arquivo ou pasta | Finalidade |
|---|---|
| `.devcontainer/devcontainer.json` | Descreve as ferramentas do ambiente remoto |
| `.devcontainer/setup.sh` | Prepara o ambiente Python do projeto |
| `.devcontainer/requirements.txt` | Fixa a versão do Ansible Core |
| `docker/` | Página HTML, Dockerfile e Compose |
| `terraform/main.tf` | Declara o recurso de arquivo local |
| `terraform/.terraform.lock.hcl` (gerado por `init`) | Registra a seleção e os hashes do provider e deve ser versionado |
| `ansible/ambiente.yml` | Declara a tarefa de configuração local |
| `.gitignore` | Exclui caches, estado e arquivos temporários do versionamento |
| `.gitattributes` | Define finais de linha LF para os arquivos de texto |

### 1.3. Abrir e conferir o ambiente

1. Na página do repositório, abra **Code → Codespaces**.
2. Confira a branch e a indicação de quem será responsável pelo uso. Clique em **Create codespace on main**, ou no nome da branch principal utilizada.
3. Aguarde a preparação automática. O pacote espera o término de `setup.sh` antes de disponibilizar a sessão.
4. No VS Code do navegador, abra **Terminal → New Terminal**. Comece na raiz do projeto, onde este `README.md` está localizado.
5. Execute uma linha por vez:

```bash
pwd
docker version
docker compose version
terraform version
ansible-playbook --version
```

O Docker deve apresentar informações de **cliente e servidor**. Terraform deve mostrar **1.14.0**, e Ansible deve mostrar **Core 2.19.3**. A configuração usa Ubuntu 24.04 e Python 3.12. Não é necessário ativar manualmente o venv no Codespace, pois o pacote inclui seu diretório de comandos no `PATH` da sessão.

Se `ansible-playbook` não for encontrado, confira se a preparação terminou. Caso ela tenha falhado, leia o erro e, após corrigir sua causa, execute `bash .devcontainer/setup.sh` na raiz. Se necessário, abra um novo terminal. Se Docker não conseguir falar com o servidor, peça ao professor para conferir a preparação do ambiente antes de seguir.

Fontes: [criar um Codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/creating-a-codespace-for-a-repository), [adicionar arquivos ao repositório](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository) e [Dev Container Features](https://containers.dev/features).

### 1.4. Uso e limites do Codespaces

O uso depende da conta, da política da organização e da disponibilidade de computação e armazenamento. Contas pessoais possuem uma franquia de uso incluído. Quando a franquia se esgota, a continuidade depende das condições de cobrança e dos limites da conta. Confira a área de uso e faturamento do GitHub antes da atividade, sem habilitar gastos adicionais apenas para concluir o exercício.

Fechar a aba do navegador não para imediatamente o Codespace. Um ambiente ativo consome computação; parado, ele continua ocupando armazenamento. Ao terminar, preserve o trabalho no repositório e use **Stop codespace**. Quando não precisar mais dele e tiver confirmado o envio dos arquivos, a exclusão libera o armazenamento ocupado pelo ambiente.

Fontes: [uso incluído](https://docs.github.com/en/codespaces/troubleshooting/troubleshooting-included-usage) e [parar e iniciar um Codespace](https://docs.github.com/en/codespaces/developing-in-a-codespace/stopping-and-starting-a-codespace).

## 2. Laboratório Docker: uma página servida pelo Nginx

**Objetivo:** distinguir imagem, contêiner, construção, execução e acesso HTTP. Comece na raiz do projeto.

### 2.1. Executar a primeira instância

```bash
docker run --detach --name primeiro-nginx --publish 8080:80 nginx:stable-alpine
docker ps
```

`--detach` mantém o processo em segundo plano. `--name` permite identificar essa instância. `8080:80` publica a porta 8080 do ambiente remoto para a porta 80 do contêiner.

No Codespaces, abra **PORTS**. Se necessário, use **Add Port**, informe `8080` e pressione Enter. Use **Open in Browser** na linha da porta e mantenha a visibilidade **Private**. O resultado esperado é a página padrão do Nginx. O endereço dessa aba alcança o ambiente remoto; `localhost:8080` digitado diretamente no navegador do computador da sala aponta para o próprio computador.

### 2.2. Observar e encerrar a primeira instância

Execute e observe cada comando antes de passar ao seguinte:

```bash
docker logs --tail 10 primeiro-nginx
docker stop primeiro-nginx
docker ps -a
docker rm primeiro-nginx
```

`stop` encerra a execução. `ps -a` inclui contêineres parados. `rm` remove a instância já parada, mas preserva a imagem. Termine esta sequência para liberar a porta 8080 antes de iniciar o projeto Compose.

### 2.3. Conferir os arquivos fornecidos

```bash
cd docker
pwd
ls
```

A partir daqui, execute os comandos Docker Compose **dentro de `docker`**. Abra os três arquivos no Explorer:

- `index.html`: contém a mensagem **Ambiente de matrículas pronto**.
- `Dockerfile`: usa `nginx:stable-alpine` e inclui a página por meio de `COPY`.
- `compose.yaml`: declara o serviço `web`, a construção e a porta `8080:80`. O contexto `build: .` é relativo à pasta que contém esse arquivo.

O projeto possui um único serviço para concentrar a atividade no fluxo de construir, iniciar e verificar.

### 2.4. Validar, construir e verificar

```bash
docker compose config
docker compose up --build -d
docker compose ps
curl --fail --show-error --max-time 5 http://localhost:8080
docker compose logs --tail 20 web
```

`config` interpreta o YAML. `up --build -d` constrói a imagem e inicia ou atualiza o serviço em segundo plano. O HTML retornado pelo `curl` deve conter a mensagem da página. Abra também a porta 8080 no navegador para verificar o conteúdo renderizado. A condição `Up` em `ps` informa execução, mas não comprova sozinha o resultado HTTP.

### 2.5. Aplicar uma mudança na página

1. No editor, altere a mensagem principal de `index.html` para **Ambiente de matrículas atualizado** e salve.
2. Observe que atualizar apenas o arquivo não altera a imagem já construída.
3. Execute:

```bash
docker compose up --build -d
curl --fail --show-error --max-time 5 http://localhost:8080
```

Atualize a aba do navegador e confira a mensagem. O Compose administra a imagem construída para o serviço. Não há uma imagem autoral publicada para baixar: a construção usa os arquivos do próprio projeto.

### 2.6. Produzir uma falha controlada e corrigir

No `compose.yaml` deste laboratório, troque apenas `8080:80` por `8080:81`. Salve e execute:

```bash
docker compose up -d
docker compose ps
curl --fail --show-error --max-time 5 http://localhost:8080
docker compose logs --tail 20 web
```

O YAML continua válido, mas a porta de destino não corresponde à porta em que esse Nginx escuta. O contêiner pode aparecer em execução enquanto o teste HTTP falha. A mensagem exata do `curl` pode variar.

Restaure `8080:80`, salve e execute novamente:

```bash
docker compose up -d
curl --fail --show-error --max-time 5 http://localhost:8080
```

Registre a causa e a evidência de recuperação.

### 2.7. Encerrar e voltar à raiz

```bash
docker compose down
docker compose ps -a
cd ..
```

O comando `down` encerra os contêineres e a rede desse projeto. Os arquivos do exercício permanecem. O projeto não usa volume de dados. Depois de `cd ..`, o terminal deve voltar à raiz do repositório.

Fontes: [imagem oficial Nginx](https://hub.docker.com/_/nginx), [Dockerfile](https://docs.docker.com/reference/dockerfile/), [Compose up](https://docs.docker.com/reference/cli/docker/compose/up/), [publicação de portas](https://docs.docker.com/engine/network/port-publishing/), [Compose down](https://docs.docker.com/reference/cli/docker/compose/down/) e [portas no Codespaces](https://docs.github.com/en/codespaces/developing-in-a-codespace/forwarding-ports-in-your-codespace).

## 3. Laboratório Terraform: ciclo de vida de um arquivo real

**Objetivo:** planejar, criar, alterar e remover um recurso local. O provider `local` cria um arquivo dentro do projeto. Não é necessária uma conta de nuvem. Todos os comandos desta seção partem da **raiz**.

### 3.1. Ler a configuração

Abra `terraform/main.tf`. O provider está fixado em `2.5.2`. O recurso `local_file.ambiente` declara:

- O caminho do arquivo: `ambiente.txt` na pasta da configuração.
- O conteúdo inicial: `AMBIENTE=homologacao`, seguido de uma quebra de linha.
- A permissão `0644`: leitura para os demais usuários e escrita para o proprietário.

O primeiro `init` gera `terraform/.terraform.lock.hcl` com a seleção e os hashes do provider. Revise e inclua esse arquivo no commit. Ele não acompanha o kit inicial, para que a turma observe sua criação. Os arquivos de estado, os planos e o arquivo gerado ficam fora do Git por meio do `.gitignore`.

### 3.2. Inicializar e validar

```bash
terraform -chdir=terraform init -no-color -input=false
terraform -chdir=terraform fmt -check -no-color
terraform -chdir=terraform validate -no-color
```

`-chdir=terraform` faz a ferramenta trabalhar na pasta da configuração sem mudar o diretório do terminal. `-no-color` retira códigos de cor da saída. `-input=false` evita perguntas interativas sobre valores de entrada.

O `init` prepara as dependências. `fmt -check` confere a formatação. `validate` verifica sintaxe e consistência interna; não cria o recurso.

### 3.3. Planejar, revisar e aplicar

```bash
terraform -chdir=terraform plan -no-color -input=false -out=plano-inicial
```

Confira o caminho e o conteúdo propostos. A primeira execução do exemplo em um ambiente sem esse recurso mostrou **1 to add, 0 to change, 0 to destroy**.

Depois de revisar, execute:

```bash
terraform -chdir=terraform apply -no-color -input=false plano-inicial
cat terraform/ambiente.txt
```

O `apply` com um plano salvo não pede outra confirmação. A decisão de executá-lo ocorre depois da revisão. O `cat` verifica o resultado real no arquivo.

### 3.4. Repetir a comparação

```bash
terraform -chdir=terraform plan -no-color -input=false
```

Sem alterar o código ou o arquivo, a referência da aula mostrou **No changes**. Explique por que o novo plano não propôs criar nem substituir o arquivo. O plan compara e propõe ações; não aplica a mudança.

### 3.5. Alterar o conteúdo

No editor, altere somente a linha `content` para:

```hcl
content = "AMBIENTE=homologacao\nVERSAO=2\n"
```

Salve e execute:

```bash
terraform -chdir=terraform plan -no-color -input=false -out=plano-mudanca
```

Com o provider utilizado, a mudança de conteúdo propôs **substituir** o arquivo. A saída de referência mostrou **1 to add, 0 to change, 1 to destroy**. Depois de compreender essa proposta:

```bash
terraform -chdir=terraform apply -no-color -input=false plano-mudanca
cat terraform/ambiente.txt
```

Confira as duas linhas no arquivo. A substituição observada pertence a este recurso e provider; outros recursos podem tratar alterações de outra forma.

### 3.6. Encerrar o recurso

```bash
terraform -chdir=terraform plan -destroy -no-color -input=false
terraform -chdir=terraform destroy -no-color
```

Revise a proposta. Neste projeto, o único recurso administrado é `local_file.ambiente`. Digite `yes` quando o `destroy` solicitar confirmação. Depois:

```bash
ls -l terraform/ambiente.txt
```

O `ls` deve informar que o arquivo não existe. Esse erro de consulta é esperado após sua remoção. O arquivo `main.tf` permanece no projeto.

Fontes: [local_file](https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/file), [init](https://developer.hashicorp.com/terraform/cli/commands/init), [validate](https://developer.hashicorp.com/terraform/cli/commands/validate), [plan](https://developer.hashicorp.com/terraform/cli/commands/plan), [apply](https://developer.hashicorp.com/terraform/cli/commands/apply) e [destroy](https://developer.hashicorp.com/terraform/cli/commands/destroy).

## 4. Laboratório Ansible: configuração, repetição e diagnóstico

**Objetivo:** observar uma tarefa que mantém o conteúdo e as permissões de um arquivo. Todos os comandos partem da **raiz**. O alvo `localhost` é o próprio ambiente remoto do laboratório.

### 4.1. Ler o playbook e conferir a sintaxe

Abra `ansible/ambiente.yml`. Localize `hosts`, `connection` e `tasks`. A tarefa `ansible.builtin.copy` usa `playbook_dir` para colocar o arquivo na pasta `ansible`.

```bash
ansible-playbook -i localhost, ansible/ambiente.yml --syntax-check
```

Mantenha a vírgula depois de `localhost`: ela identifica a lista de hosts fornecida na linha de comando. O `--syntax-check` verifica a estrutura sem executar a tarefa.

### 4.2. Executar pela primeira e pela segunda vez

```bash
ansible-playbook -i localhost, ansible/ambiente.yml
cat ansible/ambiente.txt
ansible-playbook -i localhost, ansible/ambiente.yml
```

Em um ambiente inicial sem o arquivo, a referência mostrou `changed=1`. Na segunda execução, sem outras mudanças, mostrou `changed=0`. Em ambas, `failed=0`. Compare os resultados e explique a relação com o estado desejado.

### 4.3. Prever uma alteração

No playbook, altere somente o conteúdo para:

```yaml
content: "AMBIENTE=homologacao\nVERSAO=2\n"
```

Salve e execute:

```bash
ansible-playbook -i localhost, ansible/ambiente.yml --check --diff
cat ansible/ambiente.txt
```

O `copy` permite prever a mudança e mostrar a diferença. Neste exemplo, o resumo pode mostrar `changed=1` como previsão, mas a leitura do arquivo ainda deve apresentar somente o conteúdo anterior. Esse resultado é específico do suporte do módulo ao modo de verificação.

### 4.4. Aplicar a alteração

```bash
ansible-playbook -i localhost, ansible/ambiente.yml
cat ansible/ambiente.txt
```

Agora o arquivo deve conter duas linhas. Compare a evidência com a leitura feita depois do `--check`.

### 4.5. Provocar uma falha controlada

No editor, altere apenas o destino da tarefa para:

```yaml
dest: "{{ playbook_dir }}/pasta-inexistente/ambiente.txt"
```

Salve e execute o playbook. Como essa pasta não existe, o exemplo deve falhar. Leia a mensagem e localize o nome da tarefa e o contador `failed`.

```bash
ansible-playbook -i localhost, ansible/ambiente.yml
```

### 4.6. Corrigir e verificar

Restaure o destino:

```yaml
dest: "{{ playbook_dir }}/ambiente.txt"
```

Execute novamente:

```bash
ansible-playbook -i localhost, ansible/ambiente.yml
cat ansible/ambiente.txt
```

A referência voltou a `failed=0` e `changed=0`, pois o arquivo correto já continha a versão desejada. Explique por que corrigir um caminho pode resolver a falha sem reconstruir todo o ambiente.

Fontes: [ansible-playbook](https://docs.ansible.com/projects/ansible/latest/cli/ansible-playbook.html), [inventário host_list](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/host_list_inventory.html), [módulo copy](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/copy_module.html) e [check e diff](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html).

## 5. Entrega em dupla e encerramento

1. Alternem os papéis de execução e revisão. Quem revisa deve explicar o efeito esperado antes de executar cada mudança.
2. Registrem quatro evidências com legenda: uma mudança aplicada, uma repetição sem mudança, uma falha controlada e sua correção. O professor pode solicitar evidências adicionais do Docker e do Terraform.
3. Salvem as alterações de código. No Source Control, revisem os arquivos e preparem somente os itens da atividade. Criem o commit e enviem ao repositório pelo fluxo combinado na disciplina, com pull request e revisão quando solicitado.
4. Confiram a atualização na página do GitHub. O `.gitignore` evita incluir os ambientes Python, o estado e planos do Terraform e os arquivos locais gerados.
5. Abram [Your Codespaces](https://github.com/codespaces), localizem o ambiente da atividade e escolham **Stop codespace**. Para excluí-lo, primeiro confirmem que o trabalho necessário já está no repositório.

## 6. Verificação do material

O ciclo de Terraform e as execuções do Ansible descritos neste roteiro foram executados em um ambiente Linux de referência, com as versões indicadas. As saídas reais fundamentam os quadros de terminal da apresentação. Caminhos absolutos, horários, duração e parte das mensagens podem variar no ambiente do aluno.

O roteiro Docker/Nginx e a configuração do Codespaces foram revisados com documentação oficial e verificação de sintaxe. **Eles não foram executados de ponta a ponta neste ambiente de preparação.** O professor deve preparar o Codespace e conferir a disponibilidade do servidor Docker antes da aula.

As tags de imagem e as versões principais das features podem receber atualizações. Este kit fixa Terraform, o provider local e Ansible Core, mas não representa uma fixação integral de todas as dependências do ambiente.

## 7. Registro das atividades realizadas pela dupla

### 7.1. Testes com Docker — Integrante 1

Durante a atividade, utilizamos o Docker Compose para executar uma página HTML por meio do servidor Nginx.

Inicialmente, conseguimos acessar a aplicação pela porta 8080 do Codespaces e visualizar a página funcionando.

Depois, provocamos uma falha alterando o mapeamento da porta de `8080:80` para `8080:81` no arquivo `compose.yaml`. Com isso, a página deixou de responder corretamente e apresentou o erro HTTP 502 no navegador.

Para resolver o problema, restauramos a configuração original `8080:80` e executamos novamente o Docker Compose. Após a correção, a aplicação voltou a funcionar.

**Evidências:**
- `evidencias/docker/docker-pagina-funcionando.jpeg`
- `evidencias/docker/docker-erro-502.jpeg`

**Conclusão:** O teste mostrou que não basta o contêiner estar em execução. O mapeamento das portas também precisa estar correto para que a aplicação seja acessível.

### 7.2. Testes com Ansible — Integrante 1

Utilizamos o Ansible para configurar automaticamente um arquivo do ambiente por meio do playbook `ansible/ambiente.yml`.

Na primeira execução, o Ansible criou o arquivo com a configuração desejada, apresentando `changed=1` e `failed=0`.

Na segunda execução, sem alterações na configuração, o resultado foi `changed=0` e `failed=0`. Isso demonstrou a idempotência, pois o Ansible identificou que o arquivo já estava no estado desejado.

Também provocamos uma falha ao modificar o caminho de destino para uma pasta inexistente. A execução apresentou `failed=1`, indicando que o diretório não existia.

Após restaurar o caminho correto no playbook, executamos novamente o Ansible e obtivemos `failed=0`, confirmando a recuperação do ambiente.

**Evidências:**
- `evidencias/ansible/ansible-mudanca-e-repeticao.png`
- `evidencias/ansible/ansible-falha.png`
- `evidencias/ansible/ansible-recuperacao.png`
- `evidencias/ansible/ansible-versao-2-aplicada.png`
- `evidencias/ansible/ansible-versao-2-repeticao.png`

**Conclusão:** O Ansible permitiu automatizar a configuração e verificar se ela já estava correta. Também foi possível identificar e corrigir uma falha sem precisar reconstruir todo o ambiente.

### 7.3. Proposta de melhoria — Integrante 1

Foi criada a branch `proposta-integrante-1` para propor uma melhoria na página Docker.

A alteração adicionou a identificação do ambiente de homologação e seu status à página HTML.

A proposta foi enviada ao GitHub por meio do Pull Request #1, revisada e aprovada pelo Integrante 2 e posteriormente incorporada à branch `main`.

### 7.4. Testes com Terraform — Integrante 2

Utilizamos o Terraform para gerenciar um arquivo local por meio da configuração declarada em terraform/main.tf.

Inicialmente, executamos os comandos de inicialização e validação. Em seguida, realizamos um planejamento que identificou a necessidade de criar um recurso.

Após aplicar o plano, o arquivo ambiente.txt foi criado com a configuração inicial AMBIENTE=homologacao.

Na execução seguinte do planejamento, o Terraform não identificou mudanças necessárias, demonstrando que o recurso correspondia à configuração declarada.

Depois, alteramos o conteúdo para incluir VERSAO=2. O Terraform identificou a necessidade de substituir o recurso, e a mudança foi aplicada com sucesso.

Por fim, executamos o comando de destruição para remover o recurso criado e confirmamos que o arquivo deixou de existir.

**Evidências:**
- `evidencias/terraform/terraform-criacao.png.jpeg`
- `evidencias/terraform/terraform-sem-mudancas.png.jpeg`
- `evidencias/terraform/terraform-versao2.png.jpeg`
- `evidencias/terraform/terraform-remocao.png.jpeg`

**Conclusão:** O Terraform permitiu planejar, criar, atualizar e remover um recurso de maneira controlada, demonstrando o funcionamento da infraestrutura como código.

### 7.5. Revisão cruzada e conclusão

### 7.5. Revisão cruzada e conclusão

Durante a atividade, dividimos as responsabilidades pelos testes e pela documentação do projeto.

O Integrante 1 ficou responsável pelos testes com Docker e Ansible e criou o Pull Request #1, propondo uma melhoria na identificação do ambiente Docker. A alteração foi revisada e aprovada pelo Integrante 2 antes de ser incorporada à branch principal.

O Integrante 2 realizou os testes com Terraform e criou o Pull Request #2, propondo a inclusão de uma identificação do responsável pelo ambiente. A proposta foi revisada e aprovada pelo Integrante 1 antes de ser incorporada à branch principal.

A realização dos testes e das revisões mostrou a importância de trabalhar com configurações versionadas, automação e colaboração. Com Docker, Ansible e Terraform, conseguimos compreender melhor como preparar ambientes, identificar alterações e corrigir problemas de forma organizada.
