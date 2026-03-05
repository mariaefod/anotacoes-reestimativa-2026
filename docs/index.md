# Arquivo reestimativa_2026.bat

```
E:
cd E:\QlikView\dcgf\reestimativa_2026
git checkout . && git pull
make r_update
make copy_exec && make build && make deploy && qv /r reestimativa_2026.qvw 2> reestimativa.log
type reestimativa.log >> logs/log.Rout
rm reestimativa.log
source venv/Scripts/activate
make sharepoint
```

# make r_update

O "make r_update" atualiza bibliotecas do R, mas aparentemente não está funcionando.

```
.PHONY: help build munge test clean r_update sharepoint

#====================================================================
r_update: logs/r_update.Rout

logs/r_update.Rout: ./config/config.yaml
	@echo "Atualizando/instalando pacotes..."
	@Rscript --verbose config/R/update_pkgs.R $@ 2> logs/r_update.Rout

sharepoint:
	@python code/python/sharepoint_utils.py
```

`.PHONY` é do Make, usada para declarar targets “falsos”. Caso haja um arquivo com o mesmo nome do comando, o Make irá considerar o comando.

```
.PHONY: help build munge test clean r_update sharepoint
```

No Makefiles, "r_update" é um alvo que depende do arquivo "logs/r_update.Rout".

```
r_update: logs/r_update.Rout
```

E este arquivo é construído a seguir, com base no arquivo "config.yaml".

```
logs/r_update.Rout: ./config/config.yaml
	@echo "Atualizando/instalando pacotes..."
	@Rscript --verbose config/R/update_pkgs.R $@ 2> logs/r_update.Rout
```

Ele checa se o "config.yaml" foi modificado ou se "r_update.Rout" não existe. Então, atualiza/instala os pacotes, conforme funções dos arquivos "update_pkgs.R" e "helper.R".

!!! warning "Depois estudar estes dois arquivos"


# make copy_exec

```
TARGETS_COPY_EXEC := $(shell Rscript --verbose config/R/get_targets_copy.R exec 2> logs/log.Rout)

#====================================================================
copy: copy_exec copy_reest copy_loa copy_ldo ## Copia a base da receita e da despesa para sua pasta local

#====================================================================
copy_exec: $(TARGETS_COPY_EXEC)

copy_exec_rec:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_desp:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_rp:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_rp_folha:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_suplementacao:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_pago_orc:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_cota:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_cred_aut_desp:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout

copy_exec_alem_credito:
	@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout
```

* **copy_exec**: garante que o alvo principal dependa de todos os "TARGETS_COPY_EXEC".

    ```
    copy_exec: $(TARGETS_COPY_EXEC)
    ```

* **TARGETS_COPY_EXEC**: retorna "copy_exec_...", conforme nomes listados com "yes" no "munge_base" do arquivo "config.yaml"

    ??? note "TARGETS_COPY_EXEC"
        Contém os nomes do "munge_base" com "copy_" no começo, e eles são selecionados com base no "yes" ou "no". Isto feito, na hora que o "copy_exec" é chamado no ".bat", ele irá apenas chamar os "copy_" desejados.

        ```
        TARGETS_COPY_EXEC := $(shell Rscript --verbose config/R/get_targets_copy.R exec 2> logs/log.Rout)
        ```

        * **$(shell ...)**: é executado quando o Makefile é lido (fase de parse, por causa do ":="), antes de construir os targets. E os targets seriam os que estão no "get_targets_copy.R".

        ??? note "get_targets_copy.R"

            O "stem" pesquisa os argumentos de linha de comando (retornados pela função "commandArgs") dentro da lista de valores verdadeiros do "munge_base" (em "config"). No caso, o argumento de linha de comando é "exec".

            ```
            TARGETS_COPY_EXEC := $(shell Rscript --verbose config/R/get_targets_copy.R exec 2> logs/log.Rout).
            ```

            O output do "get_targets_copy.R" seria como "copy_target", por exemplo "copy_exec_rec", caso "exec_rec" esteja como "yes".

            ```
            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "congif.yaml"

            stem <- names(config$munge_base)[as.logical(config$munge_base)]    # lista os nomes com "yes" no "munge_base"

            args <- commandArgs(trailingOnly = TRUE)    # retorna argumentos de linha de comando (no caso, "exec")

            regex <- paste0("^", args, "_")    # concatena "^exec_"

            stem <- stem[grepl(regex, stem)]    # pesquisa quem começa com "exec_" na lista do "munge_base" com "yes"

            if(is.character(stem) & length(stem) == 0) {    # se "stem" for "character", mas vier zerado
              targets <- ""    # retorna que "targets" é vazio
            } else {    # em outros casos
              targets <- paste0("copy_", stem)    # coloca "copy_" no começo do nome
            }

            write(targets, file = stdout())    # escreve os targets no console
            ```

            ??? note "munge_base"
                ```
                munge_base:
                  ldo_rec: no
                  ldo_desp: no
                  loa_rec: yes
                  loa_desp: yes
                  ploa_rec: no
                  ploa_desp: no
                  reest_rec: yes
                  reest_desp: yes
                  exec_rec: yes
                  exec_desp: yes
                  exec_rp: yes
                  exec_rp_folha: yes
                  exec_suplementacao: yes
                  exec_pago_orc: yes
                  exec_cota: yes
                  exec_cred_aut_desp: yes
                  exec_alem_credito: yes
                ```

        * **Rscript --verbose**: roda o script R "config/R/get_targets_copy.R" com a flag "--verbose" (que faz o Rscript escrever mensagens detalhadas — normalmente no "stderr").

        * **2> logs/log.Rout**: redireciona o "stderr" do processo para o arquivo de log.

* **copy_exec_rec (exemplo)**: baixa o arquivo "exec_rec.xlsx" do e-mail na pasta "data-row", além de limpar filtro e formatação da planilha.

    ??? note "copy_exec_exec"
        Caso o "copy_exec_rec" seja um target, ele será listado no "copy_exec".

        Assim, irá executar o script R "copy.R", que salva o arquivo "exec_rec.xlsx" na pasta "data-row", oriundo do e-mail mais recente recebido.

        Esse arquivo passa ainda por um tratamento de limpar seus filtros e formatação.

        ```
        @Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout
        ```

        * **$@**: chama ele próprio (copy_exec_rec)

        ??? note "copy.R"
            O "copy.R", especialmente para o "copy_exec" e mais ainda para o "copy_exec_rec" (exemplo), irá buscar pelo arquivo "exec_rec.xlsx" "mais recente" nos email da "dcgce.seplag@gmail.com" e irá salvar o seu respectivo anexo na pasta "data-row".

            Além disso, usa a função "clean_excel()" para limpar os filtros e a formatação desse arquivo.

            ```
            suppressPackageStartupMessages(library(gmailr))    # usa a biblioteca gmailr
            source("config/R/helper_copy.R")    # usa as funções auxiliares do arquivo "helper_copy"

            arg <- commandArgs(trailingOnly = TRUE)    # retorna argumentos de linha de comando (no caso, "copy_exec_rec")
            base <- guess_base(arg)    # retorna a palavra depois do primeiro "_" (no caso, "exec")
            yml <- yaml::yaml.load_file("config/config.yaml")$copy[[base]]    # lê a subchave "exec:" da chave "copy:" no arquivo "congif.yaml"

            if(yml$dir == "gmail") {    # se o "dir:" for "gmail"
              copy_gmail(yml$files[[arg]])    # usa a função "copy_gmail()" na lista de "files:" (ex.: exec_rec.xlsx)
            } else if(grepl("^@dcaf", yml$dir)) {    # se o "dir:" não for gmail, mas começar com @dcaf
              copy_scppo(yml$files[[arg]], yml$dir)    # usa a função copy_scppo()
            } else {    # se não for gmail nem @dcaf (ou seja, for diretório local)
              copy_local(yml$files[[arg]], yml$dir)    # usa a função copy_local
            }

            clean_excel(from = yml$files[[arg]], to = make_path(arg))    # limpa filtro e formatação do .xlsx
            ```

            ??? note "guess_base()"
                Essa função seleciona a segunda palavra, considerando uma separação por "_" da variável.

                ```
                guess_base <- function(x) {
                  tokens <- strsplit(x, "_")
                  tokens[[1]][2]
                }
                ```

            ??? note "copy_email()"
                Essa função busca, na caixa de entrada, o nome do "file" no assunto (ex.: "exec_rec.xlsx") e se "has:attachment".

                O e-mail com a data mais recente (a data que consta no assunto, exemplo "2026-03-03" dentro de "dados-aramazem-siafi-exec_rec-2026-03-03") é selecionado.

                O anexo deste e-mail é salvo na pasta "data-raw".

                ```
                copy_gmail <- function(file) {

                  # o arquivo está na pasta do computador. Deve estar seguindo orientações como de: gmailr.r-lib.org/dev/articles/oauth-client.html
                  gm_auth_configure(path = "E:/QlikView/Siconv/json.json")

                  # autoriza o "gmailr" a visualizar e gerenciar os projetos do Gmail, declarando qual identidade do Google usar
                  gm_auth(email='dcgce.seplag@gmail.com')

                  filename <- paste0("filename:", file)    # concatena "filename: exec_rec.xlsx"
                  has_attachment <- "has:attachment"
                  query <- paste(filename, has_attachment)    # concatena "filename: exec_rec.xlsx"

                  # pega o ID de uma lista de mensagens, filtradas pelo nome do arquivo (ex.: "exec_rec.xlsx") e se "has:attachment" (se tem anexo)
                  msgs_id <- gm_id(gm_messages(search = query))

                  if(length(msgs_id) == 0) {    # se o "msgs_id" estiver zerado

                    # retorna mensagem falando que não encontrou nenhum e-mail
                    stop(paste0("Base ", file," nao encontrada no gmail."), call. = FALSE)
                  }

                  # avisa que está baixando o arquivo (do email)
                  cat(paste0("Baixando ", file, "...\n"))

                  # retorna uma lista com o assunto da mensagem
                  msgs_date <- lapply(msgs_id, function(id) {

                              # busca os metadados da mensagem e extrai o assunto
                              gm_subject(gm_message(id, format = "metadata"))

                              })

                  # atribui o respectivo nome (IDs) aos elementos da lista (assuntos), permitindo acessar o assunto por ID
                  names(msgs_date) <- msgs_id

                  # transforma a lista em caracteres, ordena em ordem decrescente (o assunto) e seleciona o 1° elemento.
                  # Por exemplo, um assunto de e-mail seria: "dados-armazem-siafi-exec_alem_credito-2026-02-27"
                  msg_id <- sort(unlist(msgs_date), decreasing = TRUE)[1]

                  # extrai uma data no formato YYYY-MM-DD do assunto (ex.: "2026-02-27")
                  msg_date = gsub(".+_(\\d{4})-(\\d{2})-(\\d{2})-.+", "\\1-\\2-\\3", as.character(msg_id))

                  # verifica se a base foi atualizada hoje e, se não estiver, emite um aviso
                  # Por exemplo, "Base exec_rec.xlsx atualizada em 2026-02-27. Base desatualizada para 2026-03-03"
                  if(msg_date != Sys.Date()){
                    warning(paste0("Base ", file, " atualizada em ", msg_date,
                                ". Base desatualizada para ", Sys.Date()), call. = FALSE)
                  }

                  # extrai a mensagem completa do e-mail com ID selecionado
                  msg <- gm_message(names(msg_id), format = "full")

                  # salva todos os anexos do e-mail selecionado no diretório "data-raw"
                  gm_save_attachments(msg, path = "data-raw")

                }
                ```

            ??? note "clean_excel()"
                Executa o VBScript "clean_excel.vbs" no "file" (ex.: exec_rec.xlsx), salvando-o, com o mesmo nome e no mesmo local, um arquivo sem filtro e sem formatação.

                Se o arquivo original e o arquivo de saída tiverem nomes diferentes, o arquivo original é excluído.

                ```
                clean_excel <- function(from, to) {
                  vbscript <- paste("cscript //nologo code/vbs/clean_excel.vbs", from, to)

                  shell(vbscript, mustWork = TRUE)    # executa o comando no terminal do Windows

                  if(from != to) {    # exclui o arquivo original se o nome de entrada for diferente do nome de saída
                    invisible(file.remove(file.path("data-raw/", from)))
                  }
                }
                ```

                ??? note "make_path()"
                    O "make_path()", no arquivo "helper_copy.R", retorna o nome do arquivo .xlsx correspondente (ex.: "exec_rec.xlsx").

                    ```
                    make_path <- function(x) {    # x seria "arg" (ex.: arg = copy_exec_rec)
                      tokens <- strsplit(x, "_")    # retorna uma lista da separação de "arg" por "_" (ex.: c(copy, exec, rec))
                      base <- tokens[[1]][2]    # retorna o segundo "token" (ex.: exec)
                      stem <- paste0(tokens[[1]][-c(1, 2)], collapse = "_")    # seleciona todos os tokens depois do segundo (ex.: rec)
                      paste0(base, "_", stem, ".xlsx")    # cria o caminho "base_stem.xlsx" (ex.: "exec_rec.xlsx")
                    }
                    ```

                ??? note "clean_excel.vbs"
                    O "vbscript" é uma linha de texto com um comando que chama o Windows Script Host para executar o VBScript "clean_excel.vbs".

                    Este script abre o excel, limpa filtros e formatação e salva no mesmo local e com o mesmo nome que o arquivo de origem.

                    ```
                    ' ------------------------------------------
                    ' Inicializacao de variaveis de sistema
                    Dim fso, wshShell

                    Set fso = CreateObject("Scripting.FileSystemObject")    # manipula arquivos e caminhos
                    Set wshShell = CreateObject("WScript.Shell")    # obtém infomrações do ambiente
                    ' ------------------------------------------
                    ' Inicializacao de variaveis de usuario
                    data_raw = fso.BuildPath(wshShell.CurrentDirectory, "\data-raw\")    # diretorio_atual\data-raw\
                    old_file = Wscript.Arguments.Item(0)    # primeiro argumento (arquivo de entrada, from)
                    old_path = fso.BuildPath(data_raw, old_file)    # combina data_raw + old_file

                    new_file = Wscript.Arguments.Item(1)    # segundo argumento (arquivo de saída, to)
                    ' ------------------------------------------
                    ' Inicializacao das variaveis do excel e opera��es
                    Dim excel, workbook

                    WScript.Echo "Limpeza data-raw\" & old_file & "..."    # escreve uma mensagem no console

                    Set excel = CreateObject("Excel.Application")    # cria uma instância do Excel
                    excel.Application.DisplayAlerts = False    # desliga alertas

                    Set workbook = excel.Workbooks.Open(fso.GetFile(old_path))    # abre o arquivo

                    On Error Resume Next
                    workbook.Worksheets("base").ShowAllData    # tenta limpar filtros ativos
                    On Error GoTo 0    # se não houver filtro, ignora o erro

                    workbook.Worksheets("base").Cells.ClearFormats    # remove todas as formatações da planilha "base"
                    workbook.SaveAs data_raw & new_file, 51    # salva o arquivo em data-raw\new_file com o formato 51 ".xlsx sem macros"
                    workbook.Close    # fecha o workbook

                    excel.Application.DisplayAlerts = True    # reativa os alertas
                    excel.Application.Quit    # fecha a aplicação Excel
                    ' ------------------------------------------
                    ' Finaliza��o
                    Set fso = Nothing    # libera os objetos
                    Set workbook = Nothing
                    Set excel = Nothing
                    Set wshShell = Nothing

                    wScript.Quit    # encerra o script
                    ```


# make build

```
.PHONY: help build munge test clean r_update sharepoint

#====================================================================
MUNGE_FILES := $(shell Rscript --verbose config/R/generate_munge_files.R 2> logs/log.Rout)
EXEC_FILES := $(shell Rscript --verbose config/R/generate_exec_files.R 2> logs/log.Rout)
TEST_FILES := $(shell Rscript --verbose config/R/generate_test_files.R 2> logs/log.Rout)
LIB := $(wildcard code/R/lib/*.R)
CONFIG := config/config.yaml

#====================================================================
build: munge data/metadados.csv data/testes.csv code/qvw/load_bases.qvs ## Atualiza make munge, testes de consistência e metadados das bases execução
	@echo
	@echo "Build completo."
	@echo "-----------------------------------------------------"

munge: $(MUNGE_FILES) ## Atualiza as bases presentes em data/ que possuem script correspondente em code/R/munge/
	@echo
	@echo "Munging completo."
	@echo "-----------------------------------------------------"

#====================================================================
$(MUNGE_FILES): data/%.csv: code/R/munge/%.R data-raw/%.xlsx $(CONFIG) $(LIB)
	@Rscript --verbose config/R/check_pkg_version.R 2> logs/log.Rout
	@echo "Atualizando $*..."
	@Rscript --verbose $< 2> logs/log.Rout

data/metadados.csv: config/R/metadados.R $(EXEC_FILES) $(CONFIG) $(LIB)
	@echo "Atualizando metadados..."
	@Rscript --verbose $< 2> logs/log.Rout

data/testes.csv: tests/test.R $(TEST_FILES) $(MUNGE_FILES) $(CONFIG) $(LIB)
	@echo "Atualizando testes..."
	@Rscript --verbose $< 2> logs/log.Rout

code/qvw/load_bases.qvs: config/R/load_bases.R $(CONFIG)
	@echo "Atualizando load_bases..."
	@Rscript --verbose $< 2> logs/log.Rout
```

* **MUNGE_FILES**: lista o caminho dos arquivos "data/%.csv", criados com base na lista "munge_base: yes" do arquivo "config.yaml"

    ??? note "MUNGE_FILES"

        Na etapa de parse, é atribuído ao `MUNGE_FILES` a execução no terminal do scrip R "generate_munge_files.R", o qual cria um .csv dos "munge_base: yes" do arquivo "config.yaml", na pasta "data".

        Além disso, direciona o "stderr" para o arquivo "log.Rout", e o "stdout" é capturado para "MUNGE_FILES".

        ```
        MUNGE_FILES := $(shell Rscript --verbose config/R/generate_munge_files.R 2> logs/log.Rout)
        ```

        ??? note "generate_munge_files.R"
            Este arquivo lista, no stdout(), todos os .csv, para as bases que estão como "yes" no "munge_base" do arquivo "config.yaml".

            ```
            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"

            # extrai nome dos arquivos que devem sofrer munge de acordo com config.yaml (que estão com "yes")
            stem <- names(config$munge_base)[as.logical(config$munge_base)]

            # concatena "data/stem.csv" (ex.: "data/exec_rec.csv")
            files <- paste0("data/", stem, ".csv")

            # escreve a lista no stdout (imprime cada caminho em uma linha)
            write(files, file = stdout())
            ```

* **EXEC_FILES**: lista o caminho dos arquivos "data-raw/%.xlsx", criados com base na lista "munge_base: exec_: yes" do arquivo "config.yaml"

    ??? note "EXEC_FILES"
        O mesmo acontece para `EXEC_FILES`, mas este executa o "generate_exec_files.R", o qual cria um .xlsx dos "munge_base: exec_: yes" do arquivo "config.yaml", na pasta "data-raw".

        ```
        EXEC_FILES := $(shell Rscript --verbose config/R/generate_exec_files.R 2> logs/log.Rout)
        ```

        ??? note "generate_exec_files.R"
            Este arquivo lista, no stdout(), todos os .xlsx, para as bases que estão como "yes" no "munge_base" e que começam com "exec_" do arquivo "config.yaml".

            ```
            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"
            stem <- names(config$munge_base)[as.logical(config$munge_base)]    # filtra as bases marcadas como ativas
            stem <- stem[grepl("^exec_", stem)]    # da lista do munge_base, seleciona apenas os que começam com "exec_"

            if(length(stem) > 0) {    # caso haja base que começa com "exec_"...
              files <- paste0("data-raw/", stem, ".xlsx")    # cria o caminho "data-raw/stem.xlsx"
            } else {    # se não houver nenhum "exec_"...
              files <- ""    # o "files" ficará vazio ""
            }

            write(files, file = stdout())    # escreve a lista no stdout
            ```

* **TEST_FILES**: lista o caminho dos arquivos "tests/test/test_*.R", criados com base na lista "test: tests: yes" + "test: base: reest" do arquivo "config.yaml"

    ??? note "TEST_FILES"
        O mesmo acontece para `TEST_FILES`, mas este executa o "generate_test_files.R", o qual cria um .R dos "test: tests: yes" do arquivo "config.yaml", além de adicionar "test_" no começo do nome. Adiciona ainda a base (base: "reest") como um desses arquivos. Esses arquivos são salvos na pasta "tests/test".

        ```
        TEST_FILES := $(shell Rscript --verbose config/R/generate_test_files.R 2> logs/log.Rout)
        ```

        ??? note "generate_test_files.R"
            Este arquivo lista, no stdout(), todos os .R, para as bases que estão como "yes" no "test: tests:" do arquivo "config.yaml". Adiciona no começo do nome o prefixo "test_".

            ```
            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"

            if(config$test$run_tests == TRUE) {    # se "test: run_tests: yes" do config.yaml...
              stem <- names(config$test$tests)[as.logical(config$test$tests)]    # filtra as bases marcadas como ativas do "test: tests:"
              stem <- c(config$test$base, stem)    # adiciona "reest" na lista ("test: base: reest")
              files <- paste0("tests/test/test_", stem, ".R")    # cria o caminho "tests/test/test_stem.R"    # ex.: "tests/test/test_reest.R"
            } else{    # se "test: run_tests:" não estiver como sim
              files <- ""    # o file ficará vazio ""
            }

            write(files, file = stdout())    # escreve a lista no stdout
            ```

* **LIB**: lista o caminho dos arquivos "code/R/lib/*.R"

    ??? note "LIB"
        No caso do `LIB`, ele lista todos os caminhos com arquivos .R dentro da pasta "code/R/lib/". Exemplo: "code/R/lib/demonstrativo_fiscal.R code/R/lib/set_criterios_desp.R code/R/lib/set_criterios_rec.R".

        ```
        LIB := $(wildcard code/R/lib/*.R)
        ```

* **CONFIG**: lista o caminho "config/config.yaml"

    ??? note "CONFIG"
        No caso do `CONFIG`, ele representa o arquivo config/config.yaml.

        ```
        CONFIG := config/config.yaml
        ```

* **$(MUNGE_FILES)**: se algum dos arquivos "code/R/munge/%.R", "data-raw/%.xlsx", "config.yaml" ou "code/R/lib/*.R" for mais novo que o "data/%.csv", instala/atualiza os pacotes exigidos, conforme versão especificada no "config.yaml"

    ??? note "$(MUNGE_FILES)"

        Os alvos são os arquivos listados pelo "MUNGE_FILES", que tenha como padrão o formato "data/%.csv".

        E, para cada um deles, tem-se como pré-requisito um script R e um .xlsx correspondente ("code/R/munge/%.R" e "data-raw/%.xlsx"), com o mesmo "stem". Além do "config.yaml" e dos arquivos "code/R/lib/*.R"

        ```
        $(MUNGE_FILES): data/%.csv: code/R/munge/%.R data-raw/%.xlsx $(CONFIG) $(LIB)
            @Rscript --verbose config/R/check_pkg_version.R 2> logs/log.Rout    # checagem de versões de pacotes R
            @echo "Atualizando $*..."    # mensagem informativa
            @Rscript --verbose $< 2> logs/log.Rout    # executa o script R específico da base
        ```

        Dessa forma, se qualquer um desses arquivos for "mais novo" que o "MUNGE_FILES" (ex.: "data/exec_rec.csv"), é executado o script R "config/R/check_pkg_version.R".

        Este script R irá conferir se o pacote está na versão especificada, e, caso não esteja e se o usuário permitir, os pacotes serão instalados, conforme "version:" e "remote:" especificados no "config.yaml".

        ??? note "check_pkg_version.R"
            Confere se os pacotes R estão instalados e se a versão instalada está correta com a versão exigida.

            Caso haja algum problema, inicia um processo de instalação/atualização dos pacotes R, nas versões e nos diretórios especificados no "config.yaml".

            O usuário deve autorizar ou não a instalação.

            Se o processo falhar, o usuário deve fazer a instalação dos pacotes.

            ```
            source("config/R/helper.R")    # carrega funções auxiliares definidas em helper.R

            update_infra_pkgs()    # instala pacotes remotes, yaml e devtools que permitem restante do fluxo

            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"

            # na chave "pkgs" do "config.yaml", estão definidos o "version" e o "remote" de cada "pkgs"
            required_versions <- lapply(config$pkgs, `[[`, "version")    # lista nomeada, em que o nome é o "pkgs" e o valor é o "version"

            # vetor com TRUE/FALSE informando se o pacote precisa ser atualizado
            update_pkg <- Map(need_update,    # retorna TRUE, se precisar instalar/atualizar, e FALSE, caso contrário
                              names(config$pkgs),    # lista os pacotes da chave "pkgs" do arquivo "config.yaml"
                              required_versions)

            # retorna o nome "pkgs", dos "pkg" que devem ser atualizados/instalados (need_update = TRUE)
            update_pkg <- names(update_pkg[update_pkg==TRUE])

            # instala os pacotes que estao desatualizados que foram autorizadas pelo usuario
            invisible(lapply(update_pkg, function(pkg) {
              if(user_auth(pkg, required_versions[[pkg]])) {    # se o usuário responder que deseja instalar/atualizar o "pkg"
                cat(paste0("  Instalando pacote ", pkg, "...\n"))    # informa que o pacote está sendo instalado
                install_pkg(pkg, # ? preciso dar detach nos pacotes antes de instalar??
                            config$pkgs[[pkg]]$version,
                            config$pkgs[[pkg]]$remote)    # instala o pacote, na versão e no remote especificados no "config.yaml"
              }
              if(need_update(pkg, config$pkgs[[pkg]]$version)) {    # após atualização, revalida com "need_update()" se está tudo certo
                cat(paste0("  Instale a versão ",config$pkgs[[pkg]]$version ," do pacote ",
                           pkg," para prosseguir.\n"))    # emite mensagem solicitando que instale a versão
                stop()    # interrompe o fluxo
              }
            }))
            ```

            ??? note "update_infra_pkgs()"
                Instala o "remotes", se não estiver instalado. Ele é utilizado para baixar e instalar pacotes R armazenados no GitHub, GitLab, Bitbucket e outros.

                Instala o "yaml", caso esteja em versão abaixo da requerida. Ele é utilizado para ler os arquivos .yaml.

                Instala o "devtools", caso esteja em versão abaixo da requerida. Ele é utilizado para desenvolver pacotes R.

                ```
                update_infra_pkgs <- function() {
                  if(!"remotes" %in% installed.packages()){    # se o pacote remotes não estiver instalado,
                    cat("Instalando remotes...\n")
                    install.packages("remotes", repos = "https://cran.r-project.org/")    # instala "remotes" a partir do CRAN
                  }

                  # Obs.: essa versão é de 2014
                  if(packageVersion("yaml") < "2.1.13") {    # se a versão do "yaml" for menor que "2.1.13"
                    cat("Instalando pacote essencial yaml...")
                    install_pkg("yaml", version = "2.1.13", remote = "cran")    # instala a versão "2.1.13" do "yaml"
                  }

                  # Obs.: essa versão é de 2016
                  if(packageVersion("devtools") < "1.11.1") {    # se a versão do "devtools" for menor que "1.11.1"
                    cat("Instalando pacote essencial devtools...")
                    install_pkg("devtools", version = "1.11.1", remote = "cran")    # instala a versão "1.11.1" do "devtools"
                  }
                }
                ```

                ??? note "install_pkg()"
                    Instala o "pkg" (pacote), na "version" (versão), que consta no "remote" (repositório) especificado.

                    Os "remote" previstos são CRAN, Bitbucket e GitHub, sendo necessário definir token/credenciais para acessar os repositórios privados.

                    ```
                    install_pkg <- function(pkg, version, remote) {

                      getwd()    # retorna o caminho do diretório atual

                      options(repos = c(CRAN = "https://cran.rstudio.com/"))    # define URL do repositório para uso pelo "update.packages()"

                      personal_token <- Sys.getenv("GITHUB_SPLOR_PAT")    # obtém os valores das variáveis ​​de ambiente

                      if (identical(personal_token, "") || is.na(personal_token)) {    # se não existe token no ambiente, tenta ler arquivo
                        token_path <- "config/github_token.txt"    # arquivo com token, ignorado pelo .gitignore

                        if (file.exists(token_path)) {    # se achar o arquivo com token
                          personal_token <- readLines(token_path, warn = FALSE)    # lê as linhas de texto de uma conexão ou uma string de caracteres
                          personal_token <- trimws(personal_token)  # remove espaços em branco à esqueda e à direita de uma string de caracteres

                          if (length(personal_token) == 0 || personal_token == "") {    # se não achar, ou estiver vazio
                            personal_token <- ""
                            message("Arquivo de token está vazio.")    # avisa que o arquivo está vazio
                          }
                        } else {    # demais casos
                          personal_token <- ""
                          message("Sem acesso aos repositórios do github: token indefinido.")    # avisa que não conseguiu acessar o token
                        }
                      }

                      # Obs.: os valores "pkg", "version" e "remote" devem ser especificados ao utilizar a função
                      switch(remote,    # escolhe o instalador confome "remote"
                             cran = remotes::install_version(pkg, version = version),    # se remote = "cran", instala a versão do pacote especificados
                             bitbucket = devtools::install_bitbucket(repo = paste0("dcgf/", pkg),    # se remote = bitbucket, especifica o repositório "username/repo"
                                                                     ref = paste0("v", version),    # referência ao git (commit, tag, branch...)
                                                                     auth_user = "dcgf-admin",    # usuário, caso o pacote esteja hospedado em repositório privado
                                                                     password = "FY9AnQkMVrRFW7aJUqt5",    # senha
                                                                     upgrade = "never",    # não encontrei esse argumento
                                                                     ask = FALSE ),    # não encontrei esse argumento
                             github = devtools::install_github(repo = paste0("splor-mg/", pkg),    # se remote = github, especifica o repositório "username/repo"
                                                                      ref = paste0("v", version),    # referência ao git (commit, tag, branch...)
                                                                      auth_token = personal_token),    # token pessoal, para acessar repositório privado
                             stop("Não é possível instalar o pacote do remote especificado."))    # interrompe a execução
                    }
                    ```
            ??? note "need_update()"
                A função retorna "TRUE" se o pacote não estiver instalado ou se a versão instalada for diferente da versão especificada. Caso contrário, retorna "FALSE". Ou seja, ela responde se o pacote precisa ser instalado/atualizado.

                ```
                need_update <- function(pkg, version) {
                  if(!is_pkg_installed(pkg)) {    # checa se o "pkg" está instalado (TRUE ou FALSE)
                    return(TRUE)    # se não estiver instalado, a função retorna TRUE
                  } else {    # caso contrário
                    return(package_version(version) != packageVersion(pkg))    # compara se a versão do "version" está diferente da versão instalada do "pkg"
                  }                                                            # retorna TRUE, se não forem iguais, e FALSE, se forem iguais
                }
                ```

                ??? note "is_pkg_installed()"
                    Verifica se o "pkg" está instalado e retorna "TRUE", se estiver instalado, ou "FALSE", caso contrário.

                    ```
                    is_pkg_installed <- function(pkg) {
                      is_installed <- tryCatch(find.package(pkg),    # retorna o caminho de instalação do "pkg", caso exista
                                               error = function(e) {    # se não existir,
                                                  return(NULL)    # retorna NULL
                                               })

                      is_installed <- ifelse(is.null(is_installed), FALSE, TRUE)    # converte o caminho em "TRUE" e o "NULL" em "FALSE"
                    }
                    ```

            ??? note "user_auth()"
                Pergunta se o usuário quer instalar ou atualizar o "pkg", retornando TRUE ou FALSE.

                ```
                user_auth <- function(pkg, required_version) {

                  if(is_pkg_installed(pkg)) {    # se o "pkg" estiver instalado

                    # pergunta se o usuário quer atualizar
                    msg <- paste0("  Sua versão do pacote ", pkg, " é ", packageVersion(pkg),
                                  "\n  A versão exigida é ", required_version,
                                  "\n  Deseja instalar? (s/n) ")
                    cat(msg)

                  } else {    # se o "pkg" não estiver instalado

                    # pergunta se o usuário quer instalar
                    msg2 <- paste0("  O pacote ", pkg, " não está instalado. \n
                                   Deseja instalar? (s/n) " )
                    cat(msg2)
                  }

                  # lê a resposta e converte para minúsculas
                  user_input <- tolower(readLines(con="stdin", 1))

                  # converte "s" para TRUE e "n" para FALSE
                  ifelse(user_input == "s", TRUE, FALSE)
                }
                ```

* **data/metadados.csv**: gera/atualiza o arquivo "data/metadados.csv" com o conteúdo "metadados" das planilhas "data-raw/exec_*.xlsx"

    ??? note "data/metadados.csv"
        Quando qualquer um dos pré-requisitos for mais novo, roda o script R "config/R/metadados.R" para gerar ou atualizar o "data/metadados.csv".

        O script R gera um arquivo .csv com o conteúdo "metadados" das planilhas "data-raw/exec_*.xlsx".

        ```
        data/metadados.csv: config/R/metadados.R $(EXEC_FILES) $(CONFIG) $(LIB)
          @echo "Atualizando metadados..."
          @Rscript --verbose $< 2> logs/log.Rout    # executa a primeira dependência ("config/R/metadados.R")
        ```

        ??? note "config/R/metadados.R"
            Cria um único arquivo .csv com o data frame da aba "metadados" de cada planilha excel "exec_*" que consta em "data-raw".

            ```
            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"
            stem <- names(config$munge_base)[as.logical(config$munge_base)]    # lista os nomes com "yes" no "munge_base"
            stem <- stem[grepl("^exec_", stem)]    # pesquisa quem começa com "exec_" na lista do "munge_base" com "yes"

            if(length(stem) > 0) {    # se houver "munge_base" com "yes"
              files <- paste0("data-raw/", stem, ".xlsx")    # concatena "data-raw/stem.xlsx" (ex.: "data-raw/exec_rec.xlsx")
              metadados <- Map(readxl::read_excel, files, "metadados")    # retorna uma lista de data frames com os valores lidos da aba "metadados" de cada planilha
              metadados <- Reduce(rbind, metadados)    # transforma em um único data frame (deixa de ser uma lista)
              metadados <- metadados[!is.na(metadados$BASE),]    # remove linhas onde a coluna "BASE" está vazia
            } else {    # se não houver
              metadados <- data.frame(BASE = "Não existem bases da execução orçamentária")    # cria um data frame "BASE", em que o valor é a string
            }

            relatorios::exporta_csv(metadados, file = "data/metadados.csv", row.names = FALSE)    # exporta "metadados.csv" para a pasta "data"
            ```

            ??? note "exporta_csv()"
                Utiliza a função "fwrite()" do pacote "data.table" para exportar o arquivo .csv, considerando o separador ";" e o decimal ",", além de desabilitar a notação científica.

                Entretanto, o que será exportado, seu nome e outras configurações são dadas no argumento do "exporta_csv()".

                Obs.: "scipen" controla quando o R decide usar notação científica.

                ```
                exporta_csv <- function(...) {
                  old_scipen <- getOption("scipen")    # salva o valor atual de scipen do R
                  options(scipen = 999)    # desabilita praticamente toda notação científica
                  data.table::fwrite(..., sep = ";", dec = ",")    # exporta usando separador ";" e decimal ","
                  on.exit(options(scipen = old_scipen))    # mesmo se der erro, o valor anterior de scipen é restaurado, então não altera permanentemente as opções globais do R
                }
                ```

* **data/testes.csv**:

    ??? note "data/testes.csv"
        Quando qualquer um dos pré-requisitos for mais novo, roda o script R "tests/test.R" para gerar ou atualizar o "data/testes.csv".

        ```
        data/testes.csv: tests/test.R $(TEST_FILES) $(MUNGE_FILES) $(CONFIG) $(LIB)
          @echo "Atualizando testes..."
          @Rscript --verbose $< 2> logs/log.Rout
        ```

        ??? note "tests/test.R"

            Obs.: os arquivos "helper_test_equal" e "helper_test_greater" possuem uma função ("test_equal()" e "test_greater_than()") que cria um data frame com os resultados do teste e empilha este data frame dentro da lista "resultado_testes".

            ```
            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"

            if(config$test$run_tests == TRUE) {    # se a subchave "run_tests" estiver como "yes"
              resultado_testes <- list() # cria uma lista que os scripts de teste irão preencher

              helper_files <- list.files("tests/test/", pattern = "^helper_")    # lista os arquivos que começam com "helper_" na pasta "tests/test/"
              helper_paths <- paste0("tests/test/", helper_files)    # escreve o caminho desses arquivos
              invisible(Map(source, helper_paths, encoding = "UTF-8"))    # dá "source()" em cada helper

              stem <- names(config$test$tests)[as.logical(config$test$tests)]    # filtra as bases marcadas como ativas do "test: tests:"
              stem <- c(config$test$base, stem)    # adiciona "reest" na lista de nomes (pois "base: reest")
              files <- paste0("tests/test/test_", stem, ".R")    # escreve o caminho desses arquivos .R
              invisible(Map(source, files, encoding = "UTF-8"))    # dá "source()" em cada "test_*.R" ativado

              output <- data.table::rbindlist(resultado_testes, fill = TRUE)    # empilha as entradas da lista "resultado_testes" em um único data.frame

            } else {    # se a subchave "run_tests" estiver como "no"
              output <- data.frame(CONTEXTO = "Os testes estao configurados para nao serem executados")    # cria um data frame "CONTEXTO", em que o valor é a string
            }

            write.csv2(output, file = "data/testes.csv", row.names = F, na = "", fileEncoding = "UTF-8")    # escreve "data/testes.csv" com os resultados dos "helper_*.R" e "test_*.R"
            ```

* **code/qvw/load_bases.qvs**:

    ??? note "code/qvw/load_bases.qvs"
        Quando qualquer um dos pré-requisitos for mais novo, roda o script R "config/R/load_bases.R" para gerar ou atualizar o "code/qvw/load_bases.qvs".

        ```
        code/qvw/load_bases.qvs: config/R/load_bases.R $(CONFIG)
          @echo "Atualizando load_bases..."
          @Rscript --verbose $< 2> logs/log.Rout
        ```

        ??? note "config/R/load_bases.R"
            ```
            config <- yaml::yaml.load_file("config/config.yaml")
            stem <- names(config$munge_base)[as.logical(config$munge_base)]

            msg_prefix <- "  LOAD *\n  FROM\n  data\\"

            msg_suffix <- ".csv\n
              (txt, utf8, embedded labels, delimiter is ';', msq);
              \n  CONCATENATE\n\n"

            output <- paste0(msg_prefix, stem, msg_suffix)

            output[length(output)] <- gsub("CONCATENATE", "", output[length(output)])

            write(output, file = "code/qvw/load_bases.qvs")
            ```


## munge

Ao executar o `make build`, o make executa tudo que for necessário para o alvo `munge`, no caso, atualizar os .csv, além de atualizar `metadados.csv`, `testes.csv` e `load_bases.qvs`.

```
build: munge data/metadados.csv data/testes.csv code/qvw/load_bases.qvs
	@echo
	@echo "Build completo."
	@echo "-----------------------------------------------------"
```

O alvo `munge`, por sua vez depende da lista `$(MUNGE_FILES)`.

```
munge: $(MUNGE_FILES)
	@echo
	@echo "Munging completo."
	@echo "-----------------------------------------------------"
```

O alvo `$(MUNGE_FILES)`, por sua vez, tem como alvo `data/%.csv`, que depende de `code/R/munge/%.R`, um script R correspondente, de `data-raw/%.xlsx`, uma planilha fonte, e de `$(CONFIG)` e `$(LIB)`, outras dependências comuns, definidas em outra parte do makefile.

```
$(MUNGE_FILES): data/%.csv: code/R/munge/%.R data-raw/%.xlsx $(CONFIG) $(LIB)
	@Rscript --verbose config/R/check_pkg_version.R 2> logs/log.Rout
	@echo "Atualizando $*..."
	@Rscript --verbose $< 2> logs/log.Rout
```


- faz um munge e outras coisas

- munge transforma para pasta "data" (o data-row?)
