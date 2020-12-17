---
title: "Recuperar XML desde Delphi"
date: 2020-12-31
draft: false
slug: recuperación-de-xml
tags: ["XML", "Delphi"]
categories: ["Memorias"]
---
En un artículo anterior vimos una solución a la generación de XML (Lenguaje de Marcas Extendido, siglas en inglés) desde el ambiente de desarrollo Borland Delphi; en este veremos como podemos capturar y utilizar dicho XML para otros usos.

Para ello nos remitiremos a la misma agencia noticiosa deportiva en la cual nos basamos anteriormente, donde se pretendía cubrir las diferentes competiciones de un evento enviando las noticias y resultados a través de una aplicación diseñada por sus programadores. Si bien habíamos enfocado el problema desde el punto de vista de los reporteros (en este caso los clientes), ahora lo haremos desde el punto de vista servidor.

Los programadores de dicha agencia decidieron que una vez enviadas las noticias y/o resultados (correos) a la agencia, tendría que haber una aplicación que se encargara de procesarlos para mostrar los datos que de ellos provenían en su sitio web. Dicha aplicación no fue más que un servicio de Windows (o daemon en Linux).

Dicho servicio se encargaba cada un tiempo determinado de revisar todos los correos que habían llegado a una cuenta específica creada al efecto. Una vez obtenidos los correos eran procesados uno a uno para extraerle la información útil y guardarla primeramente en un fichero independiente y luego en la base de datos de la cual se nutre el sitio web. Para ello se confeccionó un componente ActiveX utilizando la tecnología Windows Script Component que brindaba entre sus métodos las facilidades para las inserciones pertinentes.

Analicemos entonces el código fuente de este componente script (safedata.wsc):

Primeramente se da la descripción general del componente: comentarios, descripción, versión, progid, etc.; luego se describen los métodos con sus respectivos parámetros que el mismo exporta: GetDataID, ReplaceDataID, CleanData y ProccessData. Por último se expone la implementación de estos métodos en el lenguaje especificado (en este caso Java Script).

Como se puede observar en la figura 1 tanto las etapas, los centros y atletas participantes, como los deportes llegan al servicio con sus nombres y no con sus respectivos identificadores, por lo que antes de tomar los datos para insertarlos en la base de datos, deben ser transformados; esto se logra a través de la llamada al método CleanData pasándole como parámetros el nombre del fichero XML con los datos y el DSN (Nombre de la Fuente de Datos, siglas en inglés).

Una vez acomodados los datos no falta más que llamar al método ProccessData pasándole el nombre del fichero XML de los datos, el del XSL (Lenguaje de Estilos Extendidos, siglas en inglés) que es el que se encargará de generar las consultas de inserción en la base de datos que referencia el DSN como tercer parámetro. Con esto no hace más que abrir el fichero XML, transformarlo con el XLS y luego ir iterando y ejecutando cada una de las consultas generadas por dicha transformación haciendo uso del XML DOM (Modelo de Objetos del Documento, siglas en inglés).

Entonces el código que haría falta a la hora de procesar cada correo sería:

``` Pascal
procedure TService1.ProcessMessage;

  var
    Safe : OleVariant;
    …

  begin
    …
    Safe := CreateOleObject('redway.safedata');
    Safe.CleanData('DSN=EventoX', 'informaciones.xml');
    Safe.ProccessData('informaciones.xml', 'informaciones.xsl', 'DSN=EventoX');
    …
  end;
```

Por supuesto que el cuerpo del correo que se está analizando debió ser salvado a priori en un fichero externo llamado “informaciones.xml” y debe existir un DSN de sistema llamado “EventoX” que referencie la base de datos del sitio web. Ver algunas de las relaciones de la base de datos en la figura 4

Solo resta aclarar que debe existir un fichero llamado “informaciones.xsl” que contenga las transformaciones a los datos (ver en la figura 2). Se tienen apenas dos transformaciones: una para las noticias y otra para el deporte baloncesto, se deberán tener tantas como de deportes se quiera brindar datos. Si a los datos expresados en el XML de la figura 1 los hubiésemos transformado con el XSL de la figura 2 hubiésemos tenido como resultado el XML de la figura 3.

No es objetivo de este artículo el cómo fueron trabajados los correos (para lo cual se utilizó también los componentes Indy) ni las transformaciones de los datos, ni la exhaustiva seguridad que debe tener un sistema como el tomado de referencia. Para concluir le informamos que el ejemplo expuesto aquí se utilizó como parte también del sistema noticioso de la V Universiada Nacional celebrada en las provincias Villa Clara y Sancti Spiritus en abril del 2001 con resultados muy favorables.

``` XML
<?xml version="1.0" encoding="ISO-8859-1"?>
<informaciones>
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
  <noticia>
    <dia>25</dia>
    <deporte>Todos</deporte>
    <titular>Adiós, adiós</titular>
    <cuerpo>
      Hoy se cierran las puertas luego de varios días de intensas y emocionantes jornadas. Los
      resultados preliminares permiten concluir en que ha sido un exitoso evento caracterizado
      por la excelente organización, la disciplina, el alto espíritu deportivo y la lucha por
      defender dignamente el nombre y la bandera de cada centro participante.
      Si se ha cumplido previsto, ello ha sido posible por la decisiva contribución de los
      participantes, en particular de los atletas...
    </cuerpo>
    <resumen>
      Llega el final de esta competición y con él las notas de despedida...
    </resumen>
  </noticia>
</informaciones>
```

Figura 1: Ejemplo de XML generado.

``` xml
<xsl:stylesheet xmlns:xsl="http://www.w3.org/TR/WD-xsl">
  <xsl:template match="/">
    <sqls>
      <xsl:apply-templates select="informaciones/baloncesto"/>
      <xsl:apply-templates select="informaciones/noticia"/>
    </sqls>
  </xsl:template>  
  <xsl:template match="baloncesto">
    <sql>
      INSERT INTO tbBaloncesto VALUES (
        <xsl:value-of select="dia"/>,
        '<xsl:value-of select="sexo"/>',
        '<xsl:value-of select="etapa"/>',
        '<xsl:value-of select="equipos/equipo[0]/centro"/>',
        <xsl:value-of select="equipos/equipo[0]/puntuacion"/>,
        '<xsl:value-of select="equipos/equipo[1]/centro"/>',
        <xsl:value-of select="equipos/equipo[1]/puntuacion"/>,
        '<xsl:value-of select="lanotador"/>',
        '<xsl:value-of select="ldefensivo"/>'
      )
    </sql>
    <br/>
    <br/>
  </xsl:template>
  <xsl:template match="noticia">
    <sql>
      INSERT INTO tbNoticias VALUES (
        <xsl:value-of select="dia"/>,
        '<xsl:value-of select="deporte"/>',
        '<xsl:value-of select="titular"/>',
        '<xsl:value-of select="cuerpo"/>',
        '<xsl:value-of select="resumen"/>'
      )
    </sql>
    <br/>
    <br/>
  </xsl:template>
</xsl:stylesheet>
```

Figura 2: Transfomaciones a los datos (informaciones.xsl).

``` XML
<sqls>
  <sql>
    INSERT INTO tbBaloncesto VALUES (21, 'M', 'Clasificatoria', 'UCLV', 85, 'UHO', 76, 'Orlando Abreu Rojas', 'Michael Hernández Martínez')
  </sql>
  <sql>
    INSERT INTO tbNoticias VALUES (25, 'Todos', 'Adiós, adiós', 'Hoy se cierran las puertas luego de varios días de intensas y emocionantes jornadas...')
  </sql>
</sqls>
```

Figura 3: Resultado de las transformaciones a los datos.

![Relaciones de las tablas en la base de datos](/xml-recovery/04.jpg)

Figura 4: Algunas relaciones de las tablas en la base de datos.

``` XML
<?xml version="1.0"?>
<package>
  <?component error="true" debug="true"?>
  <comment>
    Componente auxiliar para la transferencia de datos en XML a una base de datos
  </comment>
  <component id="safedata">
    <registration progid="redway.safedata" description="Transfiere los datos en XML a una base de datos" version="1.0" />
    <public>
      <method name="GetDataID">
        <parameter name="DSN"/>
        <parameter name="Table"/>
        <parameter name="IndexField"/>
        <parameter name="ValueField"/>
        <parameter name="Value"/>
      </method>
      <method name="ReplaceDataID">
        <parameter name="DSN"/>
        <parameter name="Table"/>
        <parameter name="IndexField"/>
        <parameter name="ValueField"/>
        <parameter name="Pattern"/>
        <parameter name="XMLDOM"/>
      </method>
      <method name="CleanData">
        <parameter name="XML"/>
        <parameter name="DSN"/>
      </method>
      <method name="ProccessData">
        <parameter name="XML"/>
        <parameter name="XSL"/>
        <parameter name="DSN"/>
      </method>
    </public>
    <script language="JScript">
      <![CDATA[
        function GetDataID(DSN, Table, IndexField, ValueField, Value)
          {
            var RecordSet = new ActiveXObject("ADODB.Recordset");
            RecordSet.CursorType = 3;
            RecordSet.LockType = 3;
            var result = -1;
            var SQL = "SELECT " + IndexField + " FROM " + Table + "WHERE ("+ValueField+"='"+Value + "')";
            RecordSet.Open(SQL, DSN);
            if(RecordSet.EoF == false)
                result = RecordSet.Fields.Item(0).Value
            RecordSet.Close();
            RecordSet = null;
            return result;
          }
        function ReplaceDataID(DSN, Table, IndexField, ValueField, Pattern, XMLDOM)
          {
            NodeList = XMLDOM.getElementsByTagName(Pattern);
            for(i = 0; i < NodeList.length; i++)
              {
                ID = GetDataID(DSN, Table, IndexField, ValueField, NodeList.item(i).text);
                if(ID != -1)
                    NodeList.item(i).text = ID
                  else
                    return(-1);
              }
            return(0);
          }
        function CleanData(XML, DSN)
          {
            var XMLDOM = new ActiveXObject("Microsoft.XMLDOM");
            XMLDOM.async = false;
            XMLDOM.load(XML);
            if(
               (ReplaceDataID(DSN, "tbEtapas", "ID", "descripcion", "etapa", XMLDOM) == -1) ||
               (ReplaceDataID(DSN, "tbEquipos", "ID", "abreviatura", "centro", XMLDOM) == -1) ||
               (ReplaceDataID(DSN, "tbAtletas", "ID", "nombre", "lanotador", XMLDOM) == -1) ||
               (ReplaceDataID(DSN, "tbAtletas", "ID", "nombre", "ldefensivo", XMLDOM) == -1) ||
               (ReplaceDataID(DSN, "tbDeportes", "ID", "nombre", "deporte", XMLDOM) == -1)
              )
                return(-1)
              else
              {
                XMLDOM.save(XML);
                return(0);
              }
          }
        function ProccessData(XML, XSL, DSN)
          {
            var XMLDOM = new ActiveXObject("Microsoft.XMLDOM");
            XMLDOM.async = false;
            XMLDOM.load(XML);
            var XSLDOM = new ActiveXObject("Microsoft.XMLDOM");
            XSLDOM.async = false;
            XSLDOM.load(XSL);
            var RESDOM = new ActiveXObject("Microsoft.XMLDOM");
            RESDOM.async = false;
            XMLDOM.transformNodeToObject(XSLDOM, RESDOM);
            var Connection = new ActiveXObject("ADODB.Connection");
            Connection.Open(DSN);
            var SQLS = RESDOM.getElementsByTagName("sql");
            for(i = 0; i < SQLS.length; i++)
              Connection.Execute(SQLS.item(i).text);
            Connection.Close();
            Connection = null;
            RESDOM = null;
            XSLDOM = null;
            XMLDOM = null;
          }
      ]]>
    </script>
  </component>
</package>
```

Listado 1: Código fuente del componente script (safedata.wsc).
