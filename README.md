# Classificador de arquivos com Jev

Aplicativo de linha de comando em Python que identifica o formato de um arquivo de texto e, se ele for compatível, solicita ao Jev uma avaliação de conteúdo potencialmente sensível.

## Requisitos

- Python 3.10 ou superior.
- Uma chave de API válida para a TypeSafe API.
- Acesso à internet para consultar o serviço Jev.

O programa usa somente a biblioteca padrão do Python; não requer instalação de dependências com `pip`.

## Configuração

Defina `TYPESAFE_API_KEY` no ambiente ou em um arquivo `.env` na mesma pasta de `classify.py`. O arquivo `.env.example` mostra o formato esperado. Opcionalmente, defina `TYPESAFE_MODEL`; o padrão é `jev-latest`.

No PowerShell, a variável abaixo vale para a sessão atual:

```powershell
$env:TYPESAFE_API_KEY = "sua-chave-de-api"
```

Nunca adicione `.env` nem uma chave real ao Git. O `.gitignore` já exclui arquivos `.env`, mantendo `.env.example` versionável.

## Uso

Execute a partir da pasta do projeto:

```powershell
python classify.py "C:\caminho\para\arquivo.txt"
```

Para salvar a saída JSON:

```powershell
python classify.py "C:\caminho\para\arquivo.txt" > probabilidades.json
```

Em caso de sucesso, o programa escreve no `stdout` um objeto JSON com probabilidades entre `0` e `1` para as categorias `medical`, `pii`, `credentials`, `financial`, `business`, `legal`, `security` e `other`. As categorias são avaliadas de forma independente, podem se sobrepor e as probabilidades não precisam somar `1`.

As categorias abrangem, entre outros exemplos, informações médicas, dados de identificação pessoal, credenciais, informações financeiras privadas, conteúdo comercial confidencial, informações jurídicas ou trabalhistas privadas e dados sigilosos de segurança. Conteúdo médico educativo também pode ser classificado como médico.

## Formatos aceitos

São aceitos arquivos de texto simples em ASCII ou UTF-8 válido, inclusive textos em português brasileiro com acentos. Um BOM UTF-8 inicial é aceito. Arquivos UTF-16, bytes que não formam UTF-8 válido, PDFs, imagens, arquivos binários e outros formatos não textuais são rejeitados. A extensão do arquivo, sozinha, não determina o formato; um PDF renomeado para `.txt` continua sendo rejeitado.

Arquivos vazios ou compostos apenas por espaços em branco e arquivos maiores que 32 KiB também são rejeitados. O limite é aplicado ao tamanho em bytes; o programa não trunca nem classifica parcialmente um arquivo.

A identificação de formato é uma avaliação do Jev e não um detector universal. Essa etapa envia ao serviço o nome do arquivo e uma amostra de até 4 KiB. Se o arquivo for reconhecido como texto compatível, o conteúdo completo é enviado em uma segunda solicitação para avaliação das categorias.

## Privacidade e limitações

O conteúdo e o nome do arquivo são enviados à TypeSafe API conforme descrito acima. Não use o programa com informações que você não tenha autorização para compartilhar com esse serviço; consulte as políticas do provedor antes de enviar dados sensíveis. O programa não grava localmente o conteúdo enviado nem inclui o texto ou a chave de API na saída normal.

As probabilidades são avaliações do modelo, não garantias de que um arquivo esteja seguro para compartilhar. Uma execução bem-sucedida significa que a avaliação foi concluída, não que o documento seja isento de informações sensíveis.

## Códigos de saída

- `0`: classificação concluída e probabilidades retornadas.
- `1`: erro de leitura, configuração ou comunicação com a API.
- `2`: entrada não compatível ou não classificável.

Erros são escritos no `stderr`; em caso de erro, o `stdout` permanece vazio.

## Testes

Execute os testes com:

```powershell
python -m unittest -v
```

Os testes devem simular as respostas do serviço e não fazem chamadas pagas à API.

## Arquivos de exemplo

- `sensitive.txt`: exemplo de conteúdo sintético para testar categorias sensíveis; não use como fonte de probabilidades esperadas.
- `non_sensitive.txt`: exemplo de texto público sem dados pessoais ou conteúdo confidencial.
- `test-fixtures/`: arquivos usados pelos testes automatizados.

Para classificar as amostras, configure a chave de API e execute os comandos acima substituindo o caminho pelo nome de cada arquivo. As probabilidades dependem da resposta do Jev; o nome do arquivo não determina o resultado.
