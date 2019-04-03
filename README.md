# fwf2csv

Converte um arquivo de tamanho fixo para CSV, com base em um layout definido.

## Instalação

Baixar a release [aqui](https://github.com/ricardovhz/fwf2csv/releases/download/0.0.1/fwf2csv) e copiar para um diretório do `PATH` (Ex.: `/usr/local/bin`)

## Executando

```bash
fwf2csv [-o|--only-records <registros>] [-i|--ignore-invalid] <layout> [<arquivo>]
```

O resultado do csv é impresso no stdout 

## Opções

 - -o|--only-records
   Imprime na saída apenas os registros especificados, podendo serem sepados por vírgula
 - -i|ignore-invalid
   Ignora os registros que não possuem layout definido no arquivo

## Arquivo de Layout

Um campo no arquivo de layout pode seguir o seguinte formato:

```bash
nome,tipo,inicio,tamanho
```

onde "tipo" pode ser um dos seguintes:

 - `alfa` - formato texto padrao
 - `num` - numerico, sem decimais
 - `dec[:decimais]` - decimal, sera divido por 10.0^decimais (padrao 2 decimais)
 - `date[:formato]` - formato de data (yyyy/MM/dd ou dd/MM/yyyy). Padrao **dd/MM/yyyy**

Pode-se definir tipos de registros, por exemplo:

```bash
$record_position:pos,tam[;pos,tam;...]

$record_id:0
campos do registro 0....

$record_id:1
campos do registro 1...

$record_id:9
campos do registro 9...
```

 - A instrução `record_position` define a posição da identificação do registro, podendo ser uma ou mais posições. 
É possível colocá-lo mais de uma vez no arquivo de layout, dando a cada tipo de registro a liberdade de definir sua posição.
A identificação do registro (para utilização na opção `--only-records`), se dá pela concatenação dos valores de registros.

 - A instrução `record_id` define que dessa linha para baixo serão definidos os campos do registro do tipo especificado.

## Exemplos

### Arquivos sem tipos de registros
Arquivo *dados.txt*
```
RICARDO   30
```

Arquivo *layout.csv*
```
nome,alfa,2,10
idade,num,12,2
```

### Arquivo com tipos de registros fixos
Arquivo *dados.txt*
```
0HEADER
1RICARDO   30
900001
```

Arquivo *layout.csv*
```
$record_position:1,1

$record_id:0
titulo,alfa,2,6

$record_id:1
nome,alfa,2,10
idade,num,12,2

$record_id:9
registros,num,2,5
```

### Arquivos com tipos de registros variaveis
Arquivo *dados.txt*
```
0HEADER
1ARICARDO   30
1BRUA TESTE 125  JARDIM TESTE        
900001
```

Arquivo *layout.csv*
```
$record_position:1,1

$record_id:0
titulo,alfa,2,6

$record_position:1,2

$record_id:1A
nome,alfa,3,10
idade,num,13,2

$record_id:1B
endereco,alfa,3,10
numero,num,13,5
bairro,alfa,18.20

$record_position:1,1

$record_id:9
registros,num,2,5
```

```
fwf2csv layout.csv dados.txt 
```

```
"titulo"
"HEADER"

==============================================

"nome","idade"
"RICARDO",30

==============================================

"endereco","numero","bairro"
"RUA TESTE",125,"JARDIM TESTE"

==============================================

"registros"
1
```

```
fwf2csv -o 1A layout.csv dados.txt 
```

```
"nome","idade"
"RICARDO",30
```

## Usando com o comando q (text-as-data)

Arquivo **dados.txt**
```
0HEADER
1ARICARDO   30
1BRUA TESTE 125  JARDIM TESTE        
1AJOSE      52
1BRUA TESTE 125  JARDIM TESTE        
1AMARIA     44
1BRUA TESTE 125  JARDIM TESTE        
900001
```

Arquivo **layout.csv** é o mesmo do exemplo anterior 

```bash
fwf2csv -o 1A layout.csv dados.txt 
```

```
"nome","idade"
"RICARDO",30
"JOSE",52
"MARIA",44
```

Pode-se usar o comando [q](http://harelba.github.io/q/) para tornar possível a execução de queries SQL nos arquivos.
Por exemplo, para encontrar a média de idades da saída anterior:

```bash
fwf2csv -o 1A layout.csv dados.txt | q -H -d',' "select avg(idade) from -"
42.0
```
