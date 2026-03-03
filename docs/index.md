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

O comando "copy_exec" chama os "TARGETS_COPY_EXEC".

```
TARGETS_COPY_EXEC := $(shell Rscript --verbose config/R/get_targets_copy.R exec 2> logs/log.Rout)
```

* `$(shell ...)` é executado quando o Makefile é lido (fase de parse, por causa do ":="), antes de construir os targets. E os targets seriam os que estão no "get_targets_copy.R".

??? note "get_targets_copy.R"

    Originalmente, o código está como:

    ```
    config <- yaml::yaml.load_file("config/config.yaml")

    stem <- names(config$munge_base)[as.logical(config$munge_base)]

    args <- commandArgs(trailingOnly = TRUE)

    regex <- paste0("^", args, "_")

    stem <- stem[grepl(regex, stem)]

    if(is.character(stem) & length(stem) == 0) {
      targets <- ""
    } else {
      targets <- paste0("copy_", stem)
    }

    write(targets, file = stdout())
    ```

    Unindo algumas informações:

    ```
    config <- yaml::yaml.load_file("config/config.yaml")

    stem <- grepl(paste0("^", commandArgs(trailingOnly = TRUE), "_"), names(config$munge_base)[as.logical(config$munge_base)])

    if(is.character(stem) & length(stem) == 0) {
      targets <- ""
    } else {
      targets <- paste0("copy_", stem)
    }

    write(targets, file = stdout())
    ```
    O "config" lê o arquivo congif.yaml.

    O "stem" pesquisa os argumentos de linha de comando (retornados pela função "commandArgs") dentro da lista de valores verdadeiros do "munge_base" (em "config"). Os argumentos de linha de comando são os do `exec` que está definido no comando "TARGETS_COPY_EXEC" (os "exec" podem ser consultados no "config.yaml")

    ```
    copy:
      (...)
      exec:
        dir: gmail
        files:
          copy_exec_rec: exec_rec.xlsx
          copy_exec_desp: exec_desp.xlsx
          copy_exec_rp: exec_rp.xlsx
          copy_exec_rp_folha: exec_rp_folha.xlsx
          copy_exec_suplementacao: exec_suplementacao.xlsx
          copy_exec_pago_orc: exec_pago_orc.xlsx
          copy_exec_cota: exec_cota.xlsx
          copy_exec_cred_aut_desp: cred_aut_desp.xlsx
          copy_exec_alem_credito: exec_alem_credito.xlsx

    #--------------------------------------------------------------------
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

    O output seria como "copy_target", por exemplo "copy_exec_rec", caso "exec_rec" esteja como "yes".

    * Observação: o "^" é para buscar os caracteres desejados apenas no começo da string (ex.: grepl("^cat", c("catapult", "location", "dog")) retorna true, false, false).

* Rscript --verbose: Roda o script R "config/R/get_targets_copy.R" com a flag "--verbose" (que faz o Rscript escrever mensagens detalhadas — normalmente no "stderr").

* 2> logs/log.Rout: redireciona o "stderr" do processo para o arquivo de log.

* **O que entendi**: o "cody_exec" vai targetar os "targets_copy_exec". Os "targets_copy_exec" contém os nomes dos files do "exec" com "copy_" no começo, e esses files são selecionados com base no "yes" ou "no" do "munge_base". Isto feito, na hora que o "copy_exec" é chamado no ".bat", ele irá apenas chamar os "copy_" desejados.

## copy_exec_rec (exemplo)

```
@Rscript --verbose config/R/copy.R $@ 2> logs/log.Rout
```

Caso o "copy_exec_rec" seja um target, ele será chamado pelo "copy_exec". Assim, irá executar o script R "copy.R".

* $@: chama ele próprio (copy_exec_rec)

??? note "copy.R"
    Originalmente, o código está como:

    ```
    suppressPackageStartupMessages(library(gmailr))
    source("config/R/helper_copy.R")

    arg <- commandArgs(trailingOnly = TRUE)
    base <- guess_base(arg)
    yml <- yaml::yaml.load_file("config/config.yaml")$copy[[base]]

    if(yml$dir == "gmail") {
    # options(httr_oob_default = TRUE)
      copy_gmail(yml$files[[arg]])
    } else if(grepl("^@dcaf", yml$dir)) {
      copy_scppo(yml$files[[arg]], yml$dir)
    } else {
      copy_local(yml$files[[arg]], yml$dir)
    }

    clean_excel(from = yml$files[[arg]], to = make_path(arg))
    ```

    * `arg`: é o nome do "copy_" (ex.: "copy_exec_rec")

        ```
        arg <- commandArgs(trailingOnly = TRUE)
        ```

    * `base`: lê o "arg" e retorna a palavra depois do primeiro "_" (ex.: "exec")

        ```
        base <- guess_base(arg)
        ```

        ??? note "guess_base()"
            Essa função seleciona a segunda palavra, considerando uma separação por "_" da variável.

            ```
            guess_base <- function(x) {
              tokens <- strsplit(x, "_")
              tokens[[1]][2]
            }
            ```

    * `yml`: lê o arquivo "config.yaml" e acessa a subchave "base" (ex.: "copy:" > "exec:")

        ```
        yml <- yaml::yaml.load_file("config/config.yaml")$copy[[base]]
        ```

    * `if`: aplica uma função, dependendo do diretório. Como no caso do copy_exec, o diretório é o gmail, vamos analisar apenas a função `copy_gmail(file)` (ex.: file = exec_rec.xlsx).

        ```
        if(yml$dir == "gmail") {    # ex.: se o "dir:" do "copy > exec" for gmail,
          copy_gmail(yml$files[[arg]])    # usa a função "copy_gmail()" na lista (ex.: copy > exec > files > copy_exec_rec, retornando: exec_rec.xlsx)
        } else if(grepl("^@dcaf", yml$dir)) {    # se o "dir:" não for gmail, mas começar com @dcaf
          copy_scppo(yml$files[[arg]], yml$dir)    # usa a função copy_scppo()
        } else {    # se não for gmail nem @dcaf (ou seja, for diretório local)
          copy_local(yml$files[[arg]], yml$dir)    # usa a função copy_local
        }
        ```

        - `copy_email()`: essa função busca, na caixa de entrada, o nome do "file" no assunto (ex.: "exec_rec.xlsx") e se "has:attachment".

            O e-mail com a data mais recente (a data que consta no assunto, exemplo "2026-03-03" dentro de "dados-aramazem-siafi-exec_rec-2026-03-03") é selecionado.

            O anexo deste e-mail é salvo na pasta "data-raw".

            ??? note "copy_email()"
                ```
                copy_gmail <- function(file) {
                  # if(file.exists('config/credentials.json'))
                  # cat('Arquivo encontrado')
                  gm_auth_configure(path = "E:/QlikView/Siconv/json.json")
                  gm_auth(email='dcgce.seplag@gmail.com')
                  filename <- paste0("filename:", file)
                  has_attachment <- "has:attachment"
                  query <- paste(filename, has_attachment)

                  msgs_id <- gm_id(gm_messages(search = query))

                  if(length(msgs_id) == 0) {
                    stop(paste0("Base ", file," nao encontrada no gmail."), call. = FALSE)
                  }
                  cat(paste0("Baixando ", file, "...\n"))
                  msgs_date <- lapply(msgs_id, function(id) {
                              gm_subject(gm_message(id, format = "metadata"))
                              })

                  names(msgs_date) <- msgs_id

                  msg_id <- sort(unlist(msgs_date), decreasing = TRUE)[1]

                  msg_date = gsub(".+_(\\d{4})-(\\d{2})-(\\d{2})-.+", "\\1-\\2-\\3", as.character(msg_id))

                  if(msg_date != Sys.Date()){
                    warning(paste0("Base ", file, " atualizada em ", msg_date,
                                ". Base desatualizada para ", Sys.Date()), call. = FALSE)
                  }

                  msg <- gm_message(names(msg_id), format = "full")

                  gm_save_attachments(msg, path = "data-raw")

                }
                ```

                * `gm_auth_configure()`: o arquivo está na pasta do computador. Deve estar seguindo as orientações de [configuração de um cliente OAuth](https://gmailr.r-lib.org/dev/articles/oauth-client.html).

                ```
                gm_auth_configure(path = "E:/QlikView/Siconv/json.json")
                ```

                * gm_auth(): autoriza o "gmailr" a visualizar e gerenciar os projetos do Gmail, declarando qual identidade do Google usar

                ```
                gm_auth(email='dcgce.seplag@gmail.com')
                ```

                * `msgs_id`: pega o ID de uma lista de mensagens, filtradas pelo nome do arquivo (ex.: "exec_rec.xlsx") e se "has:attachment" (ou seja, se tem anexo)

                ```
                msgs_id <- gm_id(gm_messages(search = paste(paste0("filename:", file), "has:attachment")))
                ```

                * `if`: retorna mensagem de erro, caso não haja "msgs_id" (ex.: "Base exec_rec.xlsx nao encontrada no gmail.")

                ```
                if(length(msgs_id) == 0) {    # se não houver email com o arquivo
                  stop(paste0("Base ", file," nao encontrada no gmail."), call. = FALSE)    # retorna mensagem de erro
                }

                cat(paste0("Baixando ", file, "...\n"))    # retorna mensagem (ex.: "Baixando exec_rec.xlsx...")
                ```

                * `msgs_date`: extrai a data do assunto do e-mail. Por exemplo, no e-mail "dados-armazem-siafi-exec_alem_credito-2026-02-27", seria extraído "2026-02-27".

                ```
                msgs_date <- lapply(msgs_id, function(id) {    # retorna uma lista com o assunto da mensagem
                            gm_subject(gm_message(id, format = "metadata"))    # busca os metadados da mensagem e extrai o assunto
                            })

                # atribui o respectivo nome (IDs) aos elementos da lista (assuntos), permitindo acessar o assunto por ID
                names(msgs_date) <- msgs_id

                # transforma a lista em caracteres, ordena em ordem decrescente (o assunto) e seleciona o 1° elemento.
                # Por exemplo, um assunto de e-mail seria: "dados-armazem-siafi-exec_alem_credito-2026-02-27"
                msg_id <- sort(unlist(msgs_date), decreasing = TRUE)[1]

                # extrai uma data no formato YYYY-MM-DD do assunto (ex.: "2026-02-27")
                msg_date = gsub(".+_(\\d{4})-(\\d{2})-(\\d{2})-.+", "\\1-\\2-\\3", as.character(msg_id))
                ```

                * `if`: verifica se a base foi atualizada hoje e, se não estiver, emite um aviso (ex.: "Base exec_rec.xlsx atualizada em 2026-02-27. Base desatualizada para 2026-03-03")

                ```
                if(msg_date != Sys.Date()){
                  warning(paste0("Base ", file, " atualizada em ", msg_date,
                              ". Base desatualizada para ", Sys.Date()), call. = FALSE)
                }
                ```

                * `gm_save_attachments()`: salva o anexo do e-mail selecionado em "data-raw"

                ```
                # extrai a mensagem completa do e-mail com ID selecionado
                msg <- gm_message(names(msg_id), format = "full")

                # salva todos os anexos da mensagem no diretório "data-raw"
                gm_save_attachments(msg, path = "data-raw")
                ```

    * `clean_excel`: executa o VBScript "clean_excel.vbs" no arquivo (ex.: exec_rec.xlsx), salvando-o, com o mesmo nome e no mesmo local, um arquivo sem filtro e sem formatação.

        Se o arquivo original e o arquivo de saída tiverem nomes diferentes, o arquivo original é exluído.

        ??? note "clean_excel()"

            Originalmente, o `clean_excel` está como:

            No arquivo "copy.R":

            ```
            clean_excel(from = yml$files[[arg]], to = make_path(arg))
            ```

            No arquivo "helper_copy.R":

            ```
            clean_excel <- function(from, to) {
              vbscript <- paste("cscript //nologo code/vbs/clean_excel.vbs", from, to)

              shell(vbscript, mustWork = TRUE)

              if(from != to) {
                invisible(file.remove(file.path("data-raw/", from)))
              }
            }
            ```

            ??? note "make_path()"
                O `make_path()`, no arquivo "helper_copy.R", retorna o nome do arquivo .xlsx correspondente (ex.: exec_rec.xlsx).

                ```
                make_path <- function(x) {    # x seria "arg", (ex.: arg = copy_exec_rec)
                  tokens <- strsplit(x, "_")    # retorna uma lista da separação de "arg" por "_" (ex.: c(copy, exec, rec))
                  base <- tokens[[1]][2]    # retorna o segundo "token" (ex.: exec)
                  stem <- paste0(tokens[[1]][-c(1, 2)], collapse = "_")    # seleciona todos os tokens depois do segundo (ex.: rec)
                  paste0(base, "_", stem, ".xlsx")    # cria o caminho "base_stem.xlsx" (ex.: exec_rec.xlsx)
                }
                ```

            * `vbscript`: é uma linha de texto com um comando que chama o Windows Script Host para executar o VBScript "clean_excel.vbs".

                Este script abre o excel, limpa filtros e formatação e salva no mesmo local e com o mesmo nome que o arquivo de origem.

                ??? note "clean_excel.vbs"

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

            * `shell`: executa o comando no terminal do Windows.

            * `if`: exclui o arquivo original se o nome de entrada for diferente do nome de saída

* Em suma, o "copy.R", especialmente para o "copy_exec" e mais ainda para o "copy_exec_rec" (exemplo), irá buscar pelo arquivo "exec_rec.xlsx" "mais recente" nos email da "dcgce.seplag@gmail.com" e irá salvar o seu respectivo anexo na pasta "data-row".

    Além disso, usa a função `clean_excel()` para limpar os filtros e a formatação.


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

* **MUNGE_FILES**: lista o caminho dos arquivos "data/*.csv", criados com base na lista "munge_base: yes" do arquivo "config.yaml"

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

* **EXEC_FILES**: lista o caminho dos arquivos "data-raw/*.xlsx", criados com base na lista "munge_base: exec_: yes" do arquivo "config.yaml"

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

* **$(MUNGE_FILES)**: se algum dos arquivos "code/R/munge/%.R", "data-raw/%.xlsx", "config.yaml" ou "code/R/lib/*.R" for mais novo que o "data/%.csv", instala/atualiza os pacotes exigidos, conforme versão especificada no "config/config.yaml"

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

        ??? note "check_pkg_version.R"
            Garante que os pacotes R exigidos estejam instalados nas versões especificadas no config.yaml.

            ```
            source("config/R/helper.R")    # carrega funções auxiliares definidas em helper.R

            update_infra_pkgs()    # instala pacotes yaml e devtools que permitem restante do fluxo

            config <- yaml::yaml.load_file("config/config.yaml")

            required_versions <- lapply(config$pkgs, `[[`, "version")

            # vetor com TRUE/FALSE informando se o pacote precisa ser atualizado
            update_pkg <- Map(need_update,
                              names(config$pkgs),
                              required_versions)

            update_pkg <- names(update_pkg[update_pkg==TRUE])

            # instala os pacotes que estao desatualizados que foram autorizadas pelo usuario
            invisible(lapply(update_pkg, function(pkg) {
              if(user_auth(pkg, required_versions[[pkg]])) {
                cat(paste0("  Instalando pacote ", pkg, "...\n"))
                install_pkg(pkg, # ? preciso dar detach nos pacotes antes de instalar??
                            config$pkgs[[pkg]]$version,
                            config$pkgs[[pkg]]$remote)
              }
              if(need_update(pkg, config$pkgs[[pkg]]$version)) {
                cat(paste0("  Instale a versão ",config$pkgs[[pkg]]$version ," do pacote ",
                           pkg," para prosseguir.\n"))
                stop()
              }
            }))
            ```

            ??? note "update_infra_pkgs"
                ```
                update_infra_pkgs <- function() {
                  if(!"remotes" %in% installed.packages()){
                    cat("Instalando remotes...\n")
                    install.packages("remotes", repos = "https://cran.r-project.org/")
                  }

                  if(packageVersion("yaml") < "2.1.13") {
                    cat("Instalando pacote essencial yaml...")
                    install_pkg("yaml", version = "2.1.13", remote = "cran")
                  }

                  if(packageVersion("devtools") < "1.11.1") {
                    cat("Instalando pacote essencial devtools...")
                    install_pkg("devtools", version = "1.11.1", remote = "cran")
                  }
                }
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
