# Criando relatórios usando o Koolreport

Exemplo de criação de relatório tipo barras

## Banco - cars
```sql
CREATE TABLE `colors` (
  `id` int NOT NULL AUTO_INCREMENT,
  `color` varchar(50) DEFAULT NULL,
  `date` date DEFAULT NULL,
  `value` int DEFAULT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO `colors` (`id`, `color`,`date`, `value`) VALUES
(1,	'preto','2005-05-25',	10000),
(2,	'preto','2005-05-28',	15000),
(3,	'preto','2005-06-15',	16000),
(4,	'branco','2005-06-15',	10000),
(5,	'branco','2005-06-15',	15000),
(6,	'branco','2005-06-16',	55000),
(7,	'vermelho','2005-07-18',	52000),
(8,	'vermelho','2005-07-18',	10000),
(9,	'vermelho','2005-07-21',	40000);
```

Precisar escolher bem os dois campos que representarão as colunas vertical e horizonta.

Aqui escolhi:

- color - horizontal
- value - vertical

Ele agrupará os registros pela cor e as mostrará na horizontal.

Também poderiamos escolher date e value, então ele agruparia pelo mês de cada data

mkdir /var/www/html/projeto

cd projeto
```sql
composer require koolreport/core
```

## Estrutura de vendor/koolreport/
```sql
koolreport/
├── packages/
├── src/
│   ├── clients/
│   ├── core/
│   ├── datasources/
│   ├── processes/
│   └── widgets/
├── tests/
└── autoload.php
```
## Requisitos

PHP 5.4 ou superior

## Usando os exemplos

Agora que você instalou já pode usar um dos exemplos da pasta exemplos.

Basta copiar, por exemplo os 3 arquivos do exemplo/CarsByDate para a pasta

vendor/koolreport

Criar o banco cars e importar o script cars.sql da pasta exemplos

Então configurar o banco no arquivo CarsByDate.php na função settings()

Então já pode chamar pelo navegador

http://localhost/projeto/vendor/koolreport

Ou então siga os passos seguintes para criar cada um dos arquivos.


## Criar relatório CarsByColor

cd vendor/koolreport/
```sql
/
├── koolreport/
├── CarsByColor.php
├── CarsByColor.view.php
└── index.php
```
### index.php
```php
<?php
// index.php: Just a bootstrap file
require_once "CarsByColor.php";

$carsByColor = new CarsByColor;
$carsByColor->run()->render();
```
### CarsByColor.php
```php
<?php
require_once dirname(__FILE__)."/../autoload.php";

use \koolreport\KoolReport;
use \koolreport\processes\Filter;
use \koolreport\processes\TimeBucket;
use \koolreport\processes\Group;
use \koolreport\processes\Limit;
use \koolreport\processes\Custom;

class CarsByColor extends KoolReport
{
    function settings()
    {
        return array(
            "dataSources"=>array(
                "cars_color"=>array(
                    "connectionString"=>"mysql:host=localhost;dbname=cars",
                    "username"=>"root",
                    "password"=>"",
                    "charset"=>"utf8"
                ),
            )
        ); 
    }

    protected function setup()
    {
        $this->src('cars_color')
        ->query("SELECT color,value FROM colors")
        ->pipe(new Custom(function($row){
            $row["color"] = strtolower($row["color"]);
            return $row;
        }))
        ->pipe(new Group(array(
            "by"=>"color",
            "sum"=>"value"
        )))
        ->pipe($this->dataStore('cars_by_color'));
    } 
}
```
### CarsByColor.view.php
```php
<?php 
    use \koolreport\widgets\koolphp\Table;
    use \koolreport\widgets\google\ColumnChart;
?>

<div class="report-content">
    <div class="text-center">
        <h1>Cars by Color</h1>
        <p class="lead">This report show cars by color</p>
    </div>

    <?php
    ColumnChart::create(array(
        "dataStore"=>$this->dataStore('cars_by_color'),  
        "columns"=>array(
            "color"=>array(
                "label"=>"Cor",
                "type"=>"string",
            ),
            "value"=>array(
                "label"=>"Valor",
                "type"=>"number",
                "prefix"=>"$",
            )
        ),
        "width"=>"100%",
    ));
    ?>

    <?php
    Table::create(array(
        "dataStore"=>$this->dataStore('cars_by_color'),
        "columns"=>array(
            "color"=>array(
                "label"=>"Cor",
                "type"=>"string",
            ),
            "value"=>array(
                "label"=>"Valor",
                "type"=>"number",
                "prefix"=>"$",
            )
        ),
        "cssClass"=>array(
            "table"=>"table table-hover table-bordered"
        )
    ));
    ?>
</div>

<hr>
<h3>Cars by color</h3>
<p>Aqui descreva o relatório</p>
```

## Visualizar

http://localhost/projeto/vendor/koolreport

## Tipos de gráficos

O tipo é setado no arquivo *.view.php

### Gráfico de baras

Incluir a biblioteca desejada no início do arquivo
```php
    use \koolreport\widgets\google\BarChart; // Gráfico de barras
```
Usando 
```php
    BarChart::create(array(
```
### Gráfico de colunas

Incluir a biblioteca desejada no início do arquivo
```php
    use \koolreport\widgets\google\ColumnChart; // Gráfico de colunas
```
Usando 
```php
    ColumnChart::create(array(
```
### Gráfico de pizza

Incluir a biblioteca desejada no início do arquivo
```php
    use \koolreport\widgets\google\PieChart; // Gráfico de pizza
```
Usando 
```php
    PieChart::create(array(
```
Ao mover o cursor do mouse sobre o gráfico recebemos mais informações.

Temos também:

- DonutChart
- AreaChart

### Mais detalhes sobre os tipos de gráficos

https://www.koolreport.com/docs/google_charts/overview/

## Mudar as cores do gráfico

Mudar a cor do gráfico de barras
```php
    <?php
    ColumnChart::create(array(
    "colorScheme"=>array(
        "#e7717d",
        "#c2cad0",
        "#c2b9b0",
        "#7e685a",
        "#afd275"),

        "dataStore"=>$this->dataStore('cars_by_color'),  
        "columns"=>array(
            "color"=>array(
                "label"=>"Cor",
                "type"=>"string",
            ),
            "value"=>array(
                "label"=>"Valor",
                "type"=>"number",
                "prefix"=>"$",
            )
        ),
        "width"=>"100%",
    ));
    ?>
```
Mudar cada a cor de cada uma das colunas
```php
    <?php
        ColumnChart::create([
            "dataSource"=>$this->dataStore("cars_by_color"),
            "columns"=>[
                "color",
                "value"=>[
                    "style"=>function($row) {
                        switch($row["color"])
                        {
                            case "branco":
                                return "color: #aaa";
                            case "preto":
                                return "color: #000";
                            case "vermelho":
                                return "color: #F00";
                        }
                    }
                ]
            ],
            "width"=>"100%",
        ]);
    ?>
```

## Repositório

https://github.com/ribafs/koolreport
