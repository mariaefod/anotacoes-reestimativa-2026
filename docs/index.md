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

O "make r_update" atualiza bibliotecas do R, mas aparentemente não está funcionando.

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

O "copy_exec" só roda no servidor, que versiona os arquivos copiados no "data-row", então não há necessidade de rodar o "copy" no computador pessoal.

* faz o download dos arquivos no email
* o servidor, todos os dias, roda o "copy_exec" e dá um push dessas bases. Como o "data-row" é versionado, ao dar um pull, já pega as bases atualizadas
* hipótese de estar configurado para pasta do servidor; necessitar de credencial
* será substituído pelo "dpm install"

O comando "copy_exec" chama os "TARGETS_COPY_EXEC".

```
TARGETS_COPY_EXEC := $(shell Rscript --verbose config/R/get_targets_copy.R exec 2> logs/log.Rout)
```

* `$(shell ...)` é executado quando o Makefile é lido (fase de parse), antes de contruir os targets. E os targets seriam os que estão no "get_targets_copy.R".

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

# copy_exec_rec (exemplo)

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

    * `if`: aplica uma função, dependendo do diretório

        ```
        if(yml$dir == "gmail") {    # ex.: se o "dir:" do "copy > exec" for gmail,
          copy_gmail(yml$files[[arg]])    # usa a função "copy_gmail()" na lista (ex.: copy > exec > files > copy_exec_rec, retornando: exec_rec.xlsx)
        } else if(grepl("^@dcaf", yml$dir)) {    # se o "dir:" não for gmail, mas começar com @dcaf
          copy_scppo(yml$files[[arg]], yml$dir)    # usa a função copy_scppo()
        } else {    # se não for gmail nem @dcaf (ou seja, for diretório local)
          copy_local(yml$files[[arg]], yml$dir)    # usa a função copy_local
        }
        ```

        - `gmail`: se o diretório for gmail, aplica a função copy_gmail() (ex.: file = exec_rec.xlsx)

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

                * `msgs_id`: pega o ID de uma lista de mensagens, filtradas pelo nome do arquivo (ex.: exec_rec.xlsx) e se "has:attachment" (ou seja, se tem anexo)

                ```
                msgs_id <- gm_id(gm_messages(search = paste(paste0("filename:", file), "has:attachment")))
                ```

    * `clean_excel`: usa a função clean_excel no exec_rec.xlsx e a função make_path

        ```
        function(from = yml$files[[arg]], to = make_path(arg)) {
          vbscript <- paste("cscript //nologo code/vbs/clean_excel.vbs", yml$files[[arg]], make_path(arg))

          shell(vbscript, mustWork = TRUE)    # digita comando no terminal

          if(yml$files[[arg]] != make_path(arg)) {
            invisible(file.remove(file.path("data-raw/", yml$files[[arg]])))
          }
        }
        ```


* make build: faz um munge e outras coisas
    - munge transforma para pasta "data" (o data-row?)

* .PHONY: é do Make, usada para declarar targets “falsos”. Caso haja um arquivo com o mesmo nome do comando, o Make irá considerar o comando. Mais abaixo, há a lista desses comandos explicada.
