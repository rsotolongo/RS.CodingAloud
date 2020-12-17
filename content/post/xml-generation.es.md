---
title: "Generar XML desde Delphi"
date: 2020-12-30
draft: false
slug: generación-de-xml
tags: ["XML", "Delphi"]
categories: ["Memorias"]
---
En números anteriores de esta revista salió un artículo acerca de cómo generar XML (Lenguaje de Marcas Extendido –siglas en inglés) proveniente de una forma de HTML (Lenguaje de Marcado de Hipertexto –siglas en inglés) desde ASP (Active Server Pages) con la ayuda del objeto del sistema de ficheros (FileSystemObject). En este artículo veremos cómo hacerlo desde Delphi aprovechando algunas de las incontables ventajas de este ambiente de desarrollo.

Recordemos primeramente que XML es un subconjunto de SGML (Lenguaje de Marcado Generalizado Estándar) y más que un simple lenguaje de marcas como su nombre sugiere es un meta-lenguaje que nos permite definir lenguajes de marcado adecuados a usos determinados y que está llamado a ser el nuevo ASCII debido a su fácil confección, transmisión y comprensión.

``` XML
<?xml version="1.0" encoding="ISO-8859-1"?>
<baloncesto>
  <dia>21</dia>
  <sexo>M</sexo>
  <etapa>Clasificatoria</etapa>
  <equipos>
    <equipo>
      <centro>UCLV</centro>
      <puntuacion>85</puntuacion>
    </equipo>
    <equipo>
      <centro>UHO</centro>
      <puntuacion>76</puntuacion>
    </equipo>
  </equipos>
  <lanotador>Orlando Abreu Rojas</lanotador>
  <ldefensivo>Michael Hernández Martínez</ldefensivo>
</baloncesto>
```

Figura 1: Ejemplo de XML.

Nos remitiremos a un ejemplo a grandes rasgos para ver cómo funciona en la práctica. Supongamos que una agencia noticiosa deportiva, se comprometió a brindar como servicio de su sitio web en Internet, todos los pormenores de una competición X cualquiera. La misma tiene reporteros en todos los eventos programados encargados de hacer llegar los detalles.

Pues bien, los programadores de dicha agencia luego de deliberar mucho acerca de cómo hacerlo, llegaron a la conclusión de que lo ideal sería hacer una aplicación que recogiera los datos pertinentes, los cuales pueden ser tanto noticias y entrevistas; como estadísticas y resultados de eventos en sí para el repositorio de artículos y tablas resúmenes respectivamente. Los mismos irían a parar al centro de comunicaciones a través del correo electrónico (buena elección si no se posee plena conexión a Internet) donde una vez allí, un servicio creado a la espera de dichos correos los haría guardarse en la base de datos de la cual se nutre el sitio web de la agencia.

Dicha aplicación fue implementada en Borland Delphi 6 y consiste en un conjunto de formas MDI que se acceden por menús (ver figura 2) donde cada una representa un modelo de resultado y/o noticia (ver figuras 3 y 4 respectivamente). Para no perder tiempo en la confección de la misma decidieron reutilizar tanto código como pudiesen. Para ello hicieron un módulo (unit) con una función cuya tarea es generar el XML de los datos partiendo de ciertas características de los componentes dispuestos en las formas.

![Menús de aplicación](/xml-generation/02.jpg)

Figura 2: Menús de la aplicación.

![Forma captadora de resultados](/xml-generation/03.jpg)

Figura 3: Forma captadora de resultados (en este caso un partido de baloncesto) y generadora del XML de la figura 1.

![Forma captadora de noticias](/xml-generation/04.jpg)

Figura 4: Forma captadora de noticias.

Según esta función basta con esparcir componentes y ajustarles ciertas propiedades, la propiedad Name corresponderá con el nombre de la etiqueta del XML y la propiedad Tag indicará el orden en el que serán tomados los componentes para la generación de sus respectivas etiquetas de XML. Cuando se quiera no tomar en cuenta algunos componentes bastará con dejarle su Tag con valor 0 (el por defecto). Podrá haber componentes con el mismo orden de prioridad sin que dejen de incluirse solo que indistintamente podrá procesarse uno primero o después; esto solo es importante a la hora de generar listas de etiquetas con un mismo nivel de profundidad. La lista de propiedades Tag no tiene por qué ser consecutiva, es decir, se podrán omitir números sin que se afecte el resultado. Los Paneles agrupan dentro de sí otros componentes al igual que los GroupBoxs agrupan dentro de sí listas de etiquetas.

En la figura 3 se colocó un Panel llamado “baloncesto” que agrupa todos los componentes que generarán etiquetas XML en orden (según propiedad Tag): el Edit “dia”, el ComboBox “sexo”, el ComboBox “etapa” y el Panel “equipos”, el ComboBox “lanotador” (líder anotador del partido) y el ComboBox “ldefensivo” (el más destacado en la defensa). A su vez dentro del Panel “equipos” se dispuso un GroupBox llamado “equipo” que agrupa un ListBox “centro” y un ListBox “puntuacion” estos con el mismo orden de prioridad (propiedad Tag, 1) ya que son listas que tienen (según el XML deseado) que estar al mismo nivel. Los botones: “Adicionar”, “Eliminar”, “Generar” y “Enviar” y el Memo en este caso no generarán XML por lo que se les dejó su Tag como 0.

Analicemos el código fuente de la unit “UMyXML”. En la inicialización del mismo se crea y da valores iniciales a una lista de cadenas (ClassNames) donde cada una de ellas corresponderá con una de las clases que al ser procesadas generarán etiquetas XML, no se incluyeron todas las clases de la VCL (Biblioteca de Componentes Visuales, siglas en inglés) porque solo se trabajará con este reducido número de ellas, es decir: (GroupBoxs, Paneles, Edits, ComboBoxs, SpinEdits, ListBoxs y Memos). A la única función que exporta (GenMyXML) se le pasa un control cualquiera que será el inicial para generar el XML. Luego este control es pasado a la función GetMyXML donde la misma es la encargada de obtenerle todos sus componentes y de ellos generar el XML según corresponda. Es precisamente en esta última función, donde en dependencia de la clase del componente que se le pase, se generará su etiqueta XML.

Entonces, cuando se presione el botón “Generar” lo que se hace además de todas las validaciones pertinentes: si el día está en el rango válido, si hay solo dos equipos con sus respectivas puntuaciones, etc.; es llamar a GenMyXML pasándole la forma en cuestión como el control inicial a ser procesado.

``` Pascal
procedure TBaloncestoForm.GenerarClick(Sender : TObject);

var
  XML : string;
  …

begin
  …// Validaciones
  XML := GenMyXML(Self);
  TotalXML := TotalXML + XML;
  …// Procesar XML para su visualización
end;
```

El proceso es análogo para todas las formas ya que GenMyXML es genérico y es precisamente esta característica fundamental la que hizo que se ahorrara un considerable tiempo de desarrollo. TotalXML es una variable común a todas las formas donde quedará el XML resultado de todas las informaciones captadas por el reportero con el fin de enviar solo un correo con todos los datos posibles. La aplicación sería acompañada de una base de datos con las tablas correspondientes a los centros que competirán, los deportistas, etc., para de ahí tomar dichos datos según le haga falta (ejemplo: líder anotador, destacado en la defensa, etapa, etc.). Una vez generado el XML solo restaría enviarlo a la agencia, esto se logra usando la opción “Enviar” del menú principal, la cual tendría una rutina asociada como la siguiente:

``` Pascal
procedure TMainForm.Enviar1Click(Sender : TObject);
  
  var
    Mail : TIdMessage;

  begin
    …
    Mail.Body.Add('<?xml version="1.0" encoding="ISO-8859-1"?>');
    Mail.Body.Add('<informaciones>');
    Mail.Body.Add(TotalXML);
    Mail.Body.Add('</informaciones>');
    …
  end;
```

Solo se puso parte de la rutina pues no es objetivo de este artículo enviar los correos; para lo cual se utilizaron los componentes Indy que vienen también con Kylix aunque también hay versiones para C++ Builder 4 y 5 y Delphi 5. Para concluir le informamos que el ejemplo expuesto aquí se utilizó como parte del sistema noticioso de la V Universiada Nacional celebrada en las provincias Villa Clara y Sancti Spiritus en abril del 2001 con resultados muy favorables. En un próximo artículo veremos cómo recuperar el XML de los correos y guardarlos en la base de datos.

``` Pascal
unit UMyXML;

interface

uses Controls;

function GenMyXML(const WinControl : TWinControl) : string;

implementation

uses
  StdCtrls, ExtCtrls, Spin, Classes;

var
  ClassNames : TStringList;

function GetMyXML(const WinControl : TWinControl; const Parent : string) : string;

  var
    i, j : integer;
    List : TList;

  function GetMaxControlNumber : integer;

    var i : integer;

    begin
      result := 0;
      for i := 0 to WinControl.ControlCount - 1 do
        if WinControl.Controls[i].Tag > result
          then result := WinControl.controls[i].Tag;
    end;

  procedure GetControls(const ControlNumber : integer; List : TList);

    var i : integer;

    begin
      List.Clear;
      for i := 0 to WinControl.ControlCount - 1 do
        if WinControl.Controls[i].Tag = ControlNumber
          then List.Add(WinControl.Controls[i]);
    end;

  function GetXMLTag(const List : TList; const Parent : string) : string;

    var
      i, j, index : integer;
      Items       : array of integer;
      obj         : TWinControl;
      flag        : boolean;

    begin
       result := '';
       if (List <> nil) and (List.Count > 0)
         then
           begin
             SetLength(Items, List.Count);
             flag := false;
             while not flag do
               begin
                 flag := true;
                 if Parent <> ''
                   then result := result + '<' + Parent + '>';
                 for i := 0 to List.Count -  1 do
                   begin
                     obj := TWinControl(List[i]);
                     index := ClassNames.IndexOf(obj.ClassName);
                     if index >= 0
                       then
                         begin
                           if index > 0
                             then result := result + '<' + obj.Name + '>';
                           case index of
                             0 : result := result + GetMyXML(TGroupBox(obj), obj.Name);
                             1 : result := result + GetMyXML(TPanel(obj), '');
                             2 : result := result + TEdit(obj).Text;
                             3 : result := result + TComboBox(obj).Text;
                             4 : result := result + TSpinEdit(obj).Text;
                             5 : begin
                                   result := result + TListBox(obj).Items[Items[i]];
                                   if Items[i] < TListBox(obj).Items.Count - 1
                                     then
                                       begin
                                         Inc(Items[i]);
                                         flag := false;
                                       end;
                                 end;
                             6 : begin
                                   for j := 0 to TMemo(obj).Lines.Count - 1 do
                                     result := result + TMemo(obj).Lines.Strings[j] + ' ';
                                 end;
                           end;
                         end;
                     if index > 0
                       then result := result + '</' + obj.Name + '>';
                   end;
                 if Parent <> ''
                   then result := result + '</' + Parent + '>';
               end;
           end;
    end;

  begin
    List := TList.Create;
    j := GetMaxControlNumber;
    for i := 1 to j do
      begin
        GetControls(i, List);
        result := result + GetXMLTag(List, Parent);
      end;
  end;

function GenMyXML(const WinControl : TWinControl) : string;

  begin
    result := GetMyXML(WinControl, '');
  end;

initialization
  ClassNames := TStringList.Create;
  ClassNames.Add('TGroupBox');
  ClassNames.Add('TPanel');
  ClassNames.Add('TEdit');
  ClassNames.Add('TComboBox');
  ClassNames.Add('TSpinEdit');
  ClassNames.Add('TListBox');
  ClassNames.Add('TMemo');

finalization
  ClassNames.Free;
end.
```
