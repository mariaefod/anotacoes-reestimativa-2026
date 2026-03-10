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

        Na etapa de parse, é atribuído ao `MUNGE_FILES` a execução no terminal do scrip R "generate_munge_files.R", o qual cria um .csv (vazio) dos "munge_base: yes" do arquivo "config.yaml", na pasta "data".

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

* **$(MUNGE_FILES)**: instala/atualiza pacotes exigidos, conforme versão especificada no "config.yaml", bem como cria os arquivos "data/%.csv", transformando os arquivos "data-raw/%.xlsx"

    ??? note "$(MUNGE_FILES)"
        Define como gerar arquivos "data/%.csv" a partir de scripts "code/R/munge/%.R" e seus insumos (pré-requisitos), usando Rscript.

        E, para cada um deles, tem-se como pré-requisito um script R e um .xlsx correspondente ("code/R/munge/%.R" e "data-raw/%.xlsx"), com o mesmo "stem", além do "config.yaml" e dos arquivos "code/R/lib/*.R".

        ```
        $(MUNGE_FILES): data/%.csv: code/R/munge/%.R data-raw/%.xlsx $(CONFIG) $(LIB)
            @Rscript --verbose config/R/check_pkg_version.R 2> logs/log.Rout    # checagem de versões de pacotes R
            @echo "Atualizando $*..."    # mensagem informativa
            @Rscript --verbose $< 2> logs/log.Rout    # executa o script R específico da base ("code/R/munge/%.R")
        ```

        Executa o script R "config/R/check_pkg_version.R".

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

        Além disso, também executa os scripts R que constam na pasta "code/R/munge/%.R".

        ??? note "reest_rec.R"
            Como exemplo, vamos analisar o script R "reest_rec.R", mas os outros scripts são parecidos.

            ```
            library(relatorios)
            source("code/R/lib/set_criterios_rec.R")
            source("code/R/lib/demonstrativo_fiscal.R")

            base <- reest::ler_reest_rec("data-raw/reest_rec.xlsx")    # lê o arquivo correspondente no "data-raw/", no caso, "reest_rec.xlsx"

            base$BASE <- "REEST"    # cria uma coluna "BASE", com valor "REEST" (para todas as linhas)
            base <- adiciona_desc_rec(base)
            base <- add_criterios_rec(base)
            base = add_de_para_receita(base)
            base$RECEITA_DESC_2 = NULL
            base[ANO >= 2018, RECEITA_COD_2 := RECEITA_COD]

            write.csv2(base, "data/reest_rec.csv", row.names = FALSE, na="", fileEncoding = "UTF-8")
            ```

            ??? note "adiciona_desc_rec()"
                ```
                adiciona_desc_rec <- function(base) {
                  base <- adiciona_desc(base, overwrite = TRUE, expand = TRUE)    # insere coluna de descrição
                  base <- add_poder(base)    # preenche/cria coluna de "UO_PODER" (legislativo, judiciario, executivo...)
                  return(base[])    # retorna o data.table materializado
                }

                add_criterios_rec <- function(base) {

                  base[, MDE := is_mde_rec(base)]    # flag de MDE
                  base[, FAPEMIG := is_fapemig_rec(base)]    # flag de FAPEMIG
                  base[, INTRA := FALSE]    # inicializa INTRA como FALSE
                  base[, INTRA := ifelse(nat(RECEITA_COD, 7), TRUE, FALSE)]    # marca TRUE quando nat(RECEITA_COD, 7) for TRUE
                  base[, TRANSITA := transita(base)]    # flag de TRANSITA
                  base[, ASPS := is_asps_rec(base)]    # flag de ASPS
                  base[, INTRA_SAUDE := is_intra_saude_rec(base)]    # flag de INTRA_SAUDE
                  base[, PRIMARIO := is_primario_rec(base)]    # flag de PRIMARIO
                  base[, FONTE_STN := is_fonte_stn_rec(base)]    # flag de FONTE_STN

                  base[, RCL := is_rcl(base)]    # flag de RCL
                  base[, RCL_PESSOAL := is_rcl_pessoal(base)]    # flag de RCL_PESSOAL
                  base[, RCL_DIVIDA := is_rcl_divida(base)]    # flag de RCL_DIVIDA
                  base[, PERDA_FUNDEB := is_perda_fundeb(base)]    # flag de PERDA_FUNDEB
                  base[, SEF := is_receita_SEF(base)]    # flag de SEF
                  base[, PREV := is_rec_prev(base)]    # flag de PREV

                  base <- add_demonst_grupo_rec(base)    # flag de PREV
                  base <- add_demonst_matriz_rec(base)
                  base <- add_demonstrativo_rec(base)
                  base <- demonstrativo_fontes(base)

                  base[, TIPO := "REC"]

                  return(base[])
                }
                ```

                * **adiciona_desc()**: Adiciona colunas descritivas às classificações orçamentárias, com base em algum arquivo "data-raw/desc/*.csv" ou nas tabelas descritivas no pacote "relatorios", conforme código e ano.

                * **add_poder()**: Identifica o poder no qual a Unidade Orçamentária está incluída.

                    Preenche/cria a coluna "UO_PODER" com um dos seguintes rótulos: LEGISLATIVO, JUDICIARIO, EXECUTIVO, MINISTERIO PUBLICO, TRIBUNAL DE CONTAS, DEFENSORIA PUBLICA, de acordo com a respectiva "UO_COD".

                * **add_demonst_grupo_rec()**:



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

        O script R gera um arquivo .csv com o conteúdo "metadados" das planilhas "data-raw/exec_*.xlsx".

        ```
        data/testes.csv: tests/test.R $(TEST_FILES) $(MUNGE_FILES) $(CONFIG) $(LIB)
          @echo "Atualizando testes..."
          @Rscript --verbose $< 2> logs/log.Rout
        ```

        ??? note "tests/test.R"
            Caso, no arquivo config.yaml", "test: run_tests: yes", executa os arquivos da pasta "tests/test/", exceto os marcados com "no" em "test: tests:".

            Além disso, cria uma lista "resultado_testes" a ser preenchida com os data.table gerados pelos arquivos de "tests/test/". Porém, o output de "test.R" é apenas uma data.table, com todas as informações unidades, escrita no arquivo "data/testes.csv".

            Obs.: os arquivos "helper_test_equal.R" e "helper_test_greater.R" possuem uma função ("test_equal()" e "test_greater_than()") que cria um data frame com os resultados do teste e empilha este data frame dentro da lista "resultado_testes".

            ```
            config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"

            if(config$test$run_tests == TRUE) {    # se a subchave "run_tests" estiver como "yes"
              resultado_testes <- list() # cria uma lista que os scripts de teste irão preencher

              helper_files <- list.files("tests/test/", pattern = "^helper_")    # lista os arquivos que começam com "helper_" na pasta "tests/test/"
              helper_paths <- paste0("tests/test/", helper_files)    # escreve o caminho desses arquivos
              invisible(Map(source, helper_paths, encoding = "UTF-8"))    # dá "source()" em cada helper (executa os arquivos)

              stem <- names(config$test$tests)[as.logical(config$test$tests)]    # filtra as bases marcadas como ativas do "test: tests:"
              stem <- c(config$test$base, stem)    # adiciona "reest" na lista de nomes (pois "base: reest")
              files <- paste0("tests/test/test_", stem, ".R")    # escreve o caminho desses arquivos .R
              invisible(Map(source, files, encoding = "UTF-8"))    # dá "source()" em cada "test_*.R" ativado (executa os arquivos)

              output <- data.table::rbindlist(resultado_testes, fill = TRUE)    # empilha as entradas da lista "resultado_testes" em um único data.table

            } else {    # se a subchave "run_tests" estiver como "no"
              output <- data.frame(CONTEXTO = "Os testes estao configurados para nao serem executados")    # cria um data frame "CONTEXTO", em que o valor é a string
            }

            write.csv2(output, file = "data/testes.csv", row.names = F, na = "", fileEncoding = "UTF-8")    # escreve "data/testes.csv" com os resultados dos "helper_*.R" e "test_*.R"
            ```

            ??? note "helper_dados.R"
                Cria as tabelas "rec" e "desp" no ambiente "base".

                Essas tabelas são criadas considerando o valor da subchave "test: base:".

                Por exemplo, caso seja "reest", irá procurar na pasta "data/" os arquivos "reest_rec.csv" e "reest_desp.csv.

                Assim, irá criar duas data.tables com o conteúdos desses .csv.

                ```
                library(reest)

                config <- yaml::yaml.load_file("config/config.yaml")    # lê o arquivo "config.yaml"
                base <- new.env()    # cria um novo ambiente usado para armazenar duas tabelas padronizadas ("rec" e "desp")

                run_dados_exec <- function() {
                  assign("rec", ler_exec_rec("data/exec_rec.csv"), envir = base)    # lê o "exec_rec.csv" como data.table, renomeando as colunas, e atribui em base$rec
                  data.table::setnames(base$rec, "VL_EFET_AJUST", "VL_REC")    # renomeia VL_EFET_AJUST → VL_REC
                  assign("desp", ler_exec_desp("data/exec_desp.csv"), envir = base)    # lê o "exec_desp.csv" como data.table, renomeando as colunas, e atribui em base$desp
                  data.table::setnames(base$desp, "VL_EMP", "VL_DESP")    # renomeia VL_EMP → VL_DESP
                }

                run_dados_ldo <- function() {
                  assign("rec", ler_reest_rec("data/ldo_rec.csv"), envir = base)    # lê o "ldo_rec.csv" como data.table e atribui em base$rec
                  data.table::setnames(base$rec, "VL_LDO_REC", "VL_REC")    # renomeia VL_LDO_REC → VL_REC
                  assign("desp", ler_reest_desp("data/ldo_desp.csv"), envir = base)    # lê o "ldo_desp.csv" como data.table e atribui em base$desp
                  data.table::setnames(base$desp, "VL_LDO_DESP", "VL_DESP")    # renomeia VL_LDO_DESP → VL_DESP
                }

                run_dados_loa <- function() {
                  assign("rec", ler_loa_rec("data/loa_rec.csv"), envir = base)    # lê o "loa_rec.csv" como data.table, renomeando as colunas, e atribui em base$rec
                  data.table::setnames(base$rec, "VL_LOA_REC", "VL_REC")    # renomeia VL_LOA_REC → VL_REC
                  assign("desp", ler_loa_desp("data/loa_desp.csv"), envir = base)    # lê o "loa_desp.csv" como data.table, renomeando as colunas, e atribui em base$desp
                  data.table::setnames(base$desp, "VL_LOA_DESP", "VL_DESP")    # renomeia VL_LOA_DESP → VL_DESP
                }

                run_dados_reest <- function() {
                  assign("rec", ler_reest_rec("data/reest_rec.csv"), envir = base)    # lê o "reest_rec.csv" como data.table e atribui em base$rec
                  data.table::setnames(base$rec, "VL_REEST_REC", "VL_REC")    # renomeia VL_REEST_REC → VL_REC
                  assign("desp", ler_reest_desp("data/reest_desp.csv"), envir = base)    # lê o "reest_desp.csv" como data.table e atribui em base$desp
                  data.table::setnames(base$desp, "VL_REEST_DESP", "VL_DESP")    # renomeia VL_REEST_DESP → VL_DESP
                }

                config$test$base <- tolower(config$test$base)    # transforma o valor da subchave "base" (no caso, "reest") em minúsculo
                stopifnot(config$test$base %in% c("exec", "ldo", "loa", "reest"))    # se a base não for "exec", "ldo", "loa" ou "reest", dispara erro
                switch (config$test$base,
                  "exec" = run_dados_exec(),    # se for "exec", lê "data/exec_rec.csv" e "data/exec_desp.csv"
                  "ldo" = run_dados_ldo(),    # se for "ldo", lê "data/ldo_rec.csv" e "data/ldo_desp.csv"
                  "loa" = run_dados_loa(),    # se for "loa", lê "loa_rec.csv" e "loa_desp.csv"
                  "reest" = run_dados_reest()    # se for "reest", lê "reest_rec.csv" e "reest_desp.csv"
                )

                lockEnvironment(base, bindings = TRUE)    # não permite mais reatribuir base$rec ou base$desp, nem criar/remover objetos nesse ambiente
                ```

                ??? note "ler_exec_rec()"
                    Altera nome das colunas e salva como "data.table".

                    Só aceita .csv ou .xlsx.

                    ```
                    ler_exec_rec <- function(path) {
                      #browser()
                      cols <- c(
                        `Ano de Exercício` = "ANO",
                        `Mês - Numérico` = "MES_COD",
                        `Unidade Orçamentária - Código` = "UO_COD",
                        `Unidade Orçamentária - Sigla` = "UO_SIGLA",
                        `Classificação Receita - Código` = "RECEITA_COD",
                        `Classificação Receita - Descrição` = "RECEITA_DESC",
                        `Fonte Recurso - Código` = "FONTE_COD",
                        `Fonte Recurso - Descrição` = "FONTE_DESC",
                        `Valor Previsto Inicial` = "VL_PREV_INICIAL",
                        `Valor Efetivado Ajustado` = "VL_EFET_AJUST"
                      )

                      excel <- grepl(pattern = "(.xlsx$)|(.xls$)|(.xlsm$)", path, ignore.case = TRUE)
                      csv <- grepl(pattern = ".csv$", path, ignore.case = TRUE)

                      if(excel) {
                        base <- data.table(read_excel(path))
                      } else if(csv) {
                        base <- data.table(read.csv2(path, stringsAsFactors = FALSE,  check.names = FALSE))
                      } else {
                        stop("Extensao invalida de arquivo.")
                      }

                      base <- check_base(base, cols)

                      return(base)
                    }
                    ```

                ??? note "ler_exec_desp()"
                    Altera nome das colunas e salva como "data.table".

                    Só aceita .csv ou .xlsx.

                    ```
                    ler_exec_desp <- function(path) {

                      cols <- c(
                        `Ano de Exercício` = "ANO",
                        `Mês - Numérico` = "MES_COD",
                        `Unidade Orçamentária - Código` = "UO_COD",
                        `Unidade Orçamentária - Sigla` = "UO_SIGLA",
                        `Poder Unid Orçamentária - Código` = "PODER_COD",
                        `Poder Unid Orçamentária - Desc` = "PODER_DESC",
                        `Função - Código` = "FUNCAO_COD",
                        `Função - Descrição` = "FUNCAO_DESC",
                        `Subfunção - Código` = "SUBFUNCAO_COD",
                        `Subfunção - Descrição` = "SUBFUNCAO_DESC",
                        `Programa - Código` = "PROGRAMA_COD",
                        `Programa - Descrição` = "PROGRAMA_DESC",
                        `Projeto_Atividade - Código` = "ACAO_COD",
                        `Projeto_Atividade - Descrição` = "ACAO_DESC",
                        `Categoria Econômica Despesa -Código` = "CATEGORIA_COD",
                        `Categoria Econômica Despesa - Desc` = "CATEGORIA_DESC",
                        `Grupo Despesa - Código` = "GRUPO_COD",
                        `Grupo Despesa - Descrição` = "GRUPO_DESC",
                        `Modalidade Aplicação - Código` = "MODALIDADE_COD",
                        `Modalidade Aplicação - Descrição` = "MODALIDADE_DESC",
                        `Elemento Despesa - Código` = "ELEMENTO_COD",
                        `Elemento Despesa - Descrição` = "ELEMENTO_DESC",
                        `Item Despesa - Código` = "ITEM_COD",
                        `Item Despesa - Descrição` = "ITEM_DESC",
                        `Elemento Item Despesa - Código` = "ELEMENTO_ITEM_COD",
                        `Elemento Item Despesa - Descrição` = "ELEMENTO_ITEM_DESC",
                        `Fonte Recurso - Código` = "FONTE_COD",
                        `Fonte Recurso - Descrição` = "FONTE_DESC",
                        `Identificador Orçamento - Código` = "IAG_COD",
                        `Procedência - Código` = "IPU_COD",
                        `Procedência - Descrição` = "IPU_DESC",
                        `Valor Despesa Empenhada` = "VL_EMP",
                        `Valor Despesa Liquidada` = "VL_LIQ",
                        `Valor Pago Financeiro` = "VL_PAGO_FIN",
                        `Valor Pago Orçamentário` = "VL_PAGO_ORC"
                      )

                      excel <- grepl(pattern = "(.xlsx$)|(.xls$)|(.xlsm$)", path, ignore.case = TRUE)
                      csv <- grepl(pattern = ".csv$", path, ignore.case = TRUE)

                      if(excel) {
                        sheets <- excel_sheets(path)
                        regex <- "\\([0-9]+\\)$"
                        matched <- grepl(regex, sheets)
                        raiz <- unique(sub(regex, "", sheets[matched]))

                        stopifnot(length(raiz) <= 1)

                        if(length(raiz)==0)
                          sheets <- sheets[1] else
                          sheets <- sheets[grepl(paste0("^",raiz), sheets)]

                        col_names <- c(list(TRUE), rep(list(FALSE), length(sheets) - 1))

                        base_raw <- Map(read_excel, path = path, sheet = sheets, na = "NA", col_names = col_names)
                        base <- rbindlist(base_raw)
                      } else if(csv) {
                        base <- data.table(read.csv2(path, stringsAsFactors = FALSE,  check.names = FALSE))
                      } else {
                        stop("Extensao de arquivo invalida.")
                      }

                      base <- check_base(base, cols)
                      return(base)
                    }
                    ```

                ??? note "ler_reest_rec()"
                    Salva como "data.table".

                    Só aceita .csv ou .xlsx.

                    ```
                    ler_reest_rec <- function(path) {

                      cols <- NA_character_ # nao e necessario ajustas nomes das colunas da base reest

                      excel <- grepl(pattern = "(.xlsx$)|(.xls$)|(.xlsm$)", path, ignore.case = TRUE)
                      csv <- grepl(pattern = ".csv$", path, ignore.case = TRUE)

                      if(excel) {
                        base <- data.table(read_excel(path, na = "NA", sheet = "base"))
                      } else if(csv) {
                        base <- data.table(read.csv2(path, stringsAsFactors = FALSE,  check.names = FALSE))
                      } else {
                        stop("Extensao invalida de arquivo.")
                      }

                      base <- check_base(base, cols)

                      return(base)
                    }
                    ```

                ??? note "ler_reest_desp"
                    Salva como "data.table".

                    Só aceita .csv ou .xlsx.

                    ```
                    ler_reest_desp <- function(path) {

                      cols <- NA_character_ # nao e necessario ajustas nomes das colunas da base reest

                      excel <- grepl(pattern = "(.xlsx$)|(.xls$)|(.xlsm$)", path, ignore.case = TRUE)
                      csv <- grepl(pattern = ".csv$", path, ignore.case = TRUE)

                      if(excel) {
                        base <- data.table(read_excel(path, na = "NA", sheet = "base"))
                      } else if(csv) {
                        base <- data.table(read.csv2(path, stringsAsFactors = FALSE,  check.names = FALSE))
                      } else {
                        stop("Extensao invalida de arquivo.")
                      }

                      base <- check_base(base, cols)

                      return(base)
                    }
                    ```

                ??? note "ler_loa_rec()"
                    Altera nome das colunas e salva como "data.table".

                    Só aceita .csv ou .xlsx.

                    ```
                    ler_loa_rec <- function(path) {

                      cols <- c(
                        `COD_UO` = "UO_COD",
                        `NOME_UO` = "UO_DESC",
                        `SIGLA_UO` = "UO_SIGLA",
                        `COD_FONTE` = "FONTE_COD",
                        `FONTE` = "FONTE_DESC",
                        `INTERPRETACAO` = "INTERPRETACAO",
                        `CATEGORIA` = "CATEGORIA_COD",
                        `SUBCATEGORIA` = "SUBCATEGORIA_COD",
                        `FONTE_CLAS` = "FONTE_REC_COD",
                        `RUBRICA` = "RUBRICA_COD",
                        `ALINEA` = "ALINEA_COD",
                        `SUBALINEA` = "SUBALINEA_COD",
                        `ITEM` = "ITEM_COD",
                        `COD_RECEITA` = "RECEITA_COD",
                        `RECEITA` = "RECEITA_DESC",
                        `INTERP_RECEITA` = "INTERP_RECEITA",
                        `VALOR UO (R$)` = "VL_LOA_REC_UO",
                        `VALOR SCPPO (R$)` = "VL_LOA_REC_SCPPO",
                        `VALOR FINAL (R$)` = "VL_LOA_REC",
                        `ANO` = "ANO",
                        `BASE LEGAL` = "BASE_LEGAL",
                        `METODOLOGIA DE CÁLCULO E PREMISSAS UTILIZADAS` = "MEMORIA_CALCULO"
                      )

                      excel <- grepl(pattern = "(.xlsx$)|(.xls$)|(.xlsm$)", path, ignore.case = TRUE)
                      csv <- grepl(pattern = ".csv$", path, ignore.case = TRUE)

                      if(excel) {
                        base <- data.table(read_excel(path))
                      } else if(csv) {
                        base <- data.table(read.csv2(path, stringsAsFactors = FALSE,  check.names = FALSE))
                      } else {
                        stop("Extensao invalida de arquivo.")
                      }

                      base <- check_base(base, cols)

                      return(base)
                    }
                    ```

                ??? note "ler_loa_desp()"
                    Altera nome das colunas e salva como "data.table".

                    Só aceita .csv ou .xlsx.

                    ```
                    ler_loa_desp <- function(path) {

                      cols <- c(
                        # colunas base qdd fiscal
                        `ANO` = "ANO",
                        `COD_ORGAO` = "ORGAO_COD",
                        `ORGAO` = "ORGAO_DESC",
                        `PODER` = "PODER_COD",
                        `SITUACAO` = "SITUACAO",
                        `COD_UO` = "UO_COD",
                        `UO` = "UO_DESC",
                        `CATEGORIA` = "CATEGORIA_COD",
                        `GRUPO_DESPESA` = "GRUPO_COD",
                        `MODALIDADE` = "MODALIDADE_COD",
                        `ELEMENTO_DESPESA` = "ELEMENTO_COD",
                        `FONTE` = "FONTE_COD",
                        `IPU` = "IPU_COD",
                        `SEQ_PROGTRAB` = "SEQ_PROGTRAB",
                        `FUNCAO` = "FUNCAO_COD",
                        `SUB_FUNCAO` = "SUBFUNCAO_COD",
                        `PROGRAMA` = "PROGRAMA_COD",
                        `IDENT_PROJATIV` = "IDENT_PROJ_ATIV_COD",
                        `PROJ_ATIV` = "PROJ_ATIV_COD",
                        `AÇÃO` = "ACAO_COD",
                        `SUB_PROJETO` = "SUBPROJETO_COD",
                        `VALOR UO (R$)` = "VL_LOA_DESP_UO",
                        `VALOR SCPPO (R$)` = "VL_LOA_DESP_SCPPO",
                        `VALOR FINAL (R$)` = "VL_LOA_DESP",
                        `IAG` = "IAG_COD",
                        `NOME_ACAO` = "ACAO_DESC",
                        `NOME_PROGRAMA` = "PROGRAMA_DESC",
                        # colunas base elemento item
                        `Código do Órgão` = "ORGAO_COD",
                        `Órgão` = "ORGAO_DESC",
                        `Código da UO` = "UO_COD",
                        `Unidade Orçamentária` = "UO_DESC",
                        `Função` = "FUNCAO_COD",
                        `Subfunção` = "SUBFUNCAO_COD",
                        `Programa` = "PROGRAMA_COD",
                        `Identificador` = "IDENT_PROJ_ATIV_COD",
                        `Projeto_Atividade` = "PROJ_ATIV_COD",
                        `Ação` = "ACAO_COD",
                        `Subprojeto` = "SUBPROJETO_COD",
                        `Categoria` = "CATEGORIA_COD",
                        `Grupo_Despesa` = "GRUPO_COD",
                        `Modalidade` = "MODALIDADE_COD",
                        `Elemento_Despesa` = "ELEMENTO_COD",
                        `Item_Despesa` = "ITEM_COD",
                        `Fonte` = "FONTE_COD",
                        `IPU` = "IPU_COD",
                        `IAG` = "IAG_COD",
                        `Descrição` = "ACAO_DESC",
                        `Valor (R$)` = "VL_LOA_DESP"
                      )

                      excel <- grepl(pattern = "(.xlsx$)|(.xls$)|(.xlsm$)", path, ignore.case = TRUE)
                      csv <- grepl(pattern = ".csv$", path, ignore.case = TRUE)

                      if(excel) {
                        base <- data.table(read_excel(path))
                      } else if(csv) {
                        base <- data.table(read.csv2(path, stringsAsFactors = FALSE,  check.names = FALSE))
                      } else {
                        stop("Extensao invalida de arquivo.")
                      }

                      base <- check_base(base, cols)

                      return(base)

                    }
                    ```

            ??? note "helper_test_equal.R"
                Valida se "x + adj_x" é igual a "y + adj_y" (sem considerar decimais).

                Formata os valores "x", "y", "adj_x", "adj_y" e "diff".

                Retorna um data.frame com as colunas "CONTEXTO", "DESC", "RESULTADO", "DIFF_MSG", "VALOR_X", "VALOR_Y", "AJUSTE_X", "AJUSTE_Y" e "DESC_AJUSTE".

                Acrescenta esse data.frame à lista "resultado_testes"

                ```
                test_equal <- function(x, y,
                                       adj_x = 0, adj_y = 0,
                                       descricao = "",
                                       contexto = "",
                                       desc_adj = "",
                                       label_x = "Receita", label_y = "Despesa"
                                      ) {

                  # Calcula resultado do teste
                  diff <- round(x + adj_x, 0) - round(y + adj_y, 0)    # soma "x"/"y" com "adj_x"/"adj_y", arredonda os resultados sem decimal e calcula diferença entre eles
                  resultado <- ifelse(diff == 0, TRUE, FALSE)    # se diff estiver zerado, define TRUE, se não, FALSE

                  # Formata valores: divide milhar por ".", decimal por ",", duas casas decimais
                  pretty_x <- format(x, big.mark = ".", decimal.mark = ",", nsmall = 2)
                  pretty_y <- format(y, big.mark = ".", decimal.mark = ",", nsmall = 2)

                  pretty_adj_x <- format(adj_x, big.mark = ".", decimal.mark = ",", nsmall = 2)
                  pretty_adj_y <- format(adj_y, big.mark = ".", decimal.mark = ",", nsmall = 2)

                  pretty_diff <- format(abs(diff), big.mark = ".", decimal.mark = ",", nsmall = 2)

                  # Cria mensagem em caso teste negativo (diff diferente de "0")
                  msg <- ifelse(diff > 0,
                                    paste(label_x, "maior que", label_y , "em R$", pretty_diff),
                                    ifelse(diff < 0,
                                            paste(label_x, "menor que", label_y , "em R$", pretty_diff), "ok"))

                  # Output: cria um data.frame com as colunas formatadas
                  output <- data.frame(CONTEXTO = contexto,
                                      DESC = descricao,
                                      RESULTADO = resultado,
                                      DIFF_MSG = msg,
                                      VALOR_X = pretty_x,
                                      VALOR_Y = pretty_y,
                                      AJUSTE_X = pretty_adj_x,
                                      AJUSTE_Y = pretty_adj_y,
                                      DESC_AJUSTE = desc_adj)

                  n <- length(resultado_testes)
                  resultado_testes[[n+1]] <<- output    # acrescenta o data.frame à lista "resultado_testes"
                }
                ```

            ??? note "helper_test_greater.R"
                Valida se "x" é maior que "y".

                Formata os valores "x", "y" e "diff".

                Retorna uma data.table com as colunas "CONTEXTO", "DESC", "RESULTADO", "DIFF_MSG", "VALOR_X" e "VALOR_Y".

                Acrescenta esse data.frame à lista "resultado_testes"

                ```
                test_greater_than <- function(x, y,
                                              descricao = "",
                                              contexto = "",
                                              label_x = "Receita", label_y = "Despesa"
                                             ) {

                  # Calcula resultado do teste
                  diff <- round(x - y, 0)    # calcula diferença entre "x" e "y" e arredonda o resultado sem decimal
                  resultado <- ifelse(diff >= 0, TRUE, FALSE)    # se diff estiver zerado, define TRUE, se não, FALSE

                  # Formata valores: divide milhar por ".", decimal por ",", duas casas decimais, sem formato científico
                  pretty_x <- format(x, big.mark = ".", decimal.mark = ",", nsmall = 2,
                                      scientific = FALSE)
                  pretty_y <- format(y, big.mark = ".", decimal.mark = ",", nsmall = 2,
                                      scientific = FALSE)

                  pretty_diff <- format(abs(diff), big.mark = ".", decimal.mark = ",", nsmall = 2,
                                        scientific = FALSE)

                  # Cria mensagem em caso teste negativo (diff negativa)
                  msg <- ifelse(diff < 0,
                                    paste(label_y, "maior que", label_x, "em R$", pretty_diff), "ok")

                  # Output: cria um data.frame com as colunas formatadas
                  output <- data.frame(CONTEXTO = contexto,
                                      DESC = descricao,
                                      RESULTADO = resultado,
                                      DIFF_MSG = msg,
                                      VALOR_X = pretty_x,
                                      VALOR_Y = pretty_y)

                  n <- length(resultado_testes)
                  resultado_testes[[n+1]] <<- output    # acrescenta o data.frame à lista "resultado_testes"
                }
                ```

            ??? note "test_complementacao.R"
                Cria vários data.tables, com a função "test_equal()", os quais são adicionados à lista "resultado_testes".

                Esses data.tables comparam os valores de receita e despesa de algumas situações para validar se a diferença entre eles é zero.

                Obs.: há mais conteúdo, mas ficaria muito grande, então peguei o primeiro exemplo apenas, pois não muda muita coisa.

                ```
                library(relatorios)
                #====================================================================
                # Receita vs Complementacao

                # Por exemplo, com base no arquivo "data/reest_rec.csv", vai somar os valores "VL_REC" dos "RECEITA_COD" desejados (que começam com "7999992114")
                x <- base$rec[nat(RECEITA_COD, 7999992114), sum(VL_REC)]

                y <- base$desp[ACAO_COD == 7009, sum(VL_DESP)]    # soma os valores "VL_DESP", cujo "ACAO_COD" seja "7009"

                # Usa a função "test_equal()" para validar se "x" é igual a "y".
                # Acrescenta um data.frame com essas informações à lista "resultado_testes"
                test_equal(x, y,    # valor a ser calculado a diff
                          label_x = "Receita (MATRIZ 799099111)",    # informações para a tabela
                          label_y = "Complementa��o (ACAO 7009)",
                          descricao = "Receita vs Complementa��o",
                          contexto = "Complementa��o")
                ```

                ??? note "nat()"
                    Retorna um vetor lógico, indicando quais "receita_cod" devem ser incluídos (TRUE).

                    Ao usar a função, serão incluídas as naturezas positivas e excluídas as negativas.

                    ```
                    nat <- function(receita_cod, ...) {
                      receita_cod <- nat_as_char(receita_cod)    # formato numério

                      naturezas <- c(...)
                      incluir <- which(naturezas > 0)    # indica com TRUE os índices onde "naturezas" é positivo
                      excluir <- which(!naturezas > 0)    # indica com TRUE os índices onde "naturezas" não é positivo
                      naturezas <- format(abs(naturezas), scientific = FALSE, trim = TRUE)    # valor absoluto de "naturezas"

                      n_list <- lapply(naturezas, nchar)    # lista nomeada, em que o nome é "naturezas" e o valor é "nchar" (número de caracteres)

                      # Por exemplo, se "naturezas" = "799099111" e "receita_cod" = c("1922990199000", "7990991113001"), então "substrings" = c("192299019", "799099111")
                      substrings <- Map(substr, list(receita_cod), 1, n_list)    # extrai os primeiros caracteres do "receita_cod" (do mesmo tamanho de "naturezas")

                      # Por exemplo, considerando "substrings" = c("192299019", "799099111") e "naturezas" = "799099111", retorna "FALSE TRUE".
                      # retorna uma lista com o vetor lógico de cada natureza (positiva) indicando se os elementos de "substrings[incluir]" estão dentro de "naturezas[incluir]"
                      testes_logicos_incluir <- Map(`%in%`, substrings[incluir], naturezas[incluir])
                      # retorna uma lista com o vetor lógico de cada natureza (negativa) indicando se os elementos de "substrings[excluir]" estão dentro de "naturezas[excluir]"
                      testes_logicos_excluir <- Map(`%in%`, substrings[excluir], naturezas[excluir])

                      # Transforma em um vetor lógico, combinando o vetor de cada natureza, com a condição de "OU".
                      # Ou seja, é um vetor lógico dizendo quais códigos devem ser incluídos e quais devem ser excluídos
                      logico_incluir <- Reduce(`|`, testes_logicos_incluir)
                      logico_excluir <- Reduce(`|`, testes_logicos_excluir)

                      if(is.null(logico_excluir))    # se não há naturezas negativas
                        logico_incluir    # retorna o vetor lógico "logico_incluir"
                      else    # se há negativas
                          logico_incluir & !logico_excluir   # retorna um vetor lógico, combinando "logico_incluir" com "!logico_excluir", com a condição de "E"
                    }   # Obs.: "!logico_excluir" inverte os valores de "logico_excluir" (TRUE vira FALSE)
                    ```

                    ??? note "nat_as_char()"
                        "Essa funcao foi adicionada para que receitas com 10 e 13 digitos possam ser armazenadas na mesma coluna sem que o numero de caracteres de cada uma seja igual. Isso é importante para que relatorios::verifica_intervalos funcione corretamente com base no ano da receita"

                        ```
                        nat_as_char <- function(x) {
                          format(as.numeric(x), scientific = FALSE, trim = TRUE)    # formato numério, sem formatação científica, justificado (remove espaços extras)
                        }
                        ```

            ??? note "Outros arquivos "test_*.R""
                Aparentemente todos seguem a mesma lógica de gerar data.tables com a validação se os valores da receita batem com os valores da despesa (por meio da função "test_equal()"), em diversas situações, cada um com seus próprios filtros.

                Entretanto, como não haviam arquivos na pasta "data/" ficou mais difícil de analisar caso a caso, para ter uma noção do que é feito em cada script R.

                Obs.: arquivo "test_ldo.R" vazio



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
