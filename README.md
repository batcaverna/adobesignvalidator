##### What are the Reference Codes?

The Reference Codes of the Brazilian Digital Signature Standard constitute
the reference implementation of the standards pertinent to the regulation of
digital signatures within the scope of the Brazilian Public Key Infrastructure
(ICP-Brazil). Its target audience are application and platform providers
that wish to support ICP-Brasil digital signatures. The Reference Codes
Codes aim to promote greater interoperability between such applications and facilitate
and facilitate the implementation of the standards, while simultaneously offering means to improve the set of standards itself.
improving the standards set itself.

##### What is its function?

To create and verify signatures with ICP-Brasil certificates according to the document
[DOC-ICP-15](https://www.gov.br/iti/pt-br/centrais-de-conteudo/doc-icp-15-v-3-0-visao-geral-sobre-assin-dig-na-icp-brasil-pdf),
regulated by the [National Institute of Information
Information Technology](https://gov.br/iti) (ITI).

##### How are they organized?

The main characteristic that permeates the Reference Codes is the strong use of
of reflection, a concept related to metaprogramming, to elaborate a component-oriented
component orientation, allowing modularization of some parts of the code base.
of the code base. More details about this approach can be read [in this
monograph](https://repositorio.ufsc.br/handle/123456789/184160).

The codebase is programmed in Java and follows the
[Maven](https://maven.apache.org), so the source code is at
`codes-reference-core/src/main/java`. Unit tests, located at
`codigos-de-referencia-core/src/test/java`, can be run with `mvn
test`. The `mvn` commands are usually run in the root folder of the source code.

Execution of the `mvn package` command automatically generates the documentation for the
documentation, located in the `docs` folder, and two deliverables in the form of
WAR (_web application archive_) files, both in the `target` folder:

* The [Conformance Verifier](https://verificador.iti.gov.br), verifier
  verifier of ICP-Brasil digital signatures, whose execution is started
  from the `IndexServlet` class. As this system is used in production
  its documentation is detailed below.
* The [Reference Signer](https://pbad.labsec.ufsc.br/signer-hom), which
  execution is started from the `SignerServlet` class.

##### How to install and run Compliance Verifier?

To run correctly, Verifier requires a minimum of 256 MB RAM and an
RAM and an x86-64 architecture microprocessor with two physical processing cores.
processing cores. However, the performance of the Verifier will be
directly proportional to the resources available on the machine, especially when
checking multiple files simultaneously. Therefore, **8 GB of
recommended **8 GB RAM and eight physical processing cores** for acceptable performance.
acceptable performance.

The Verifier package takes up about **35 MB** on disk. In addition, digital certificate
In addition, digital certificate files and revoked certificate lists (CRLs) are
stored locally to avoid repeated file transfers,
and thus evaluate digital signatures more quickly. In this context, the folder
folder of choice is `/tmp/compliance-checker`, and it is suggested that about **1 GB
of disk space** should be reserved for the extended use of this functionality.

The Reference Codes require **Java 15** (OpenJDK, Eclipse J9 etc.)
running under any GNU/Linux distribution (Debian, openSUSE etc.) in order to
run correctly. As Verifier is a web application, an
application server that implements the **Jakarta Servlet 5.0** standard (Apache
Tomcat 10.0+, GlassFish 6+) is also required.

The [Homologation Server](https://pbad.labsec.ufsc.br/verifier-hom) of the
Verifier uses **Ubuntu 20.04 LTS, OpenJDK 15 and [Apache Tomcat
10](https://tomcat.apache.org)**, for example. However, you can run
Verifier in other ways. The command below uses the platform
[Docker](https://docker.com) platform to run a local instance of Compliance Verifier
compiled to the path `/path/to/verifier.war`,
making it available at http://localhost:8080/verifier-docker.

```
docker run -it -p 8080:8080 tomcat:10.0.5-jdk15-adoptopenjdk-openj9 \
  -v /path/to/verifier.war:/usr/local/tomcat/webapps/verifier-docker.war
```

Verifier dependencies are automatically obtained in its
packaging process, and can be listed explicitly with the `mvn
dependency:tree` command, or checked into `pom.xml` files. The Verifier accepts
digitally signed files as its input, whether they have signatures attached, detached
attached, detached or embedded signatures. As output, it produces PDF or
HTML reports containing various information about their validity.

To install Verifier, the process is dependent on the application server
server chosen. In the case of Apache Tomcat, simply copy the WAR file to the web
web applications folder of the application server (`$CATALINA_BASE/webapps`, where
`$CATALINA_BASE` depends on the [installation method
installation method](https://tomcat.apache.org/tomcat-10.0-doc/introduction.html) of
Apache Tomcat).

Information about the signature verification process is logged
through the application server, for example in `$CATALINA_BASE/logs` in the case
Apache Tomcat. Problems in the operation of the Verifier will be exposed
These logs can be sent to the developers of the Reference
Reference Code developers as it helps with possible fixes in the code base.

The Verifier needs to download the artifacts used in the process of digital signature
digital signature verification process through the HTTP, HTTPS
and LDAP protocols. Therefore, communication through these protocols by the Verifier must not
be prohibited by the operating system or other applications.

##### Can Verifier verify signatures from other PCIs?

No. The Conformance Verifier verifies signatures according to a set of trust anchors
trust anchors, which are used to check the certification path of the
path through the certificate extension [_Authority Information
Access_](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.2.1). At
ICP-Brasil, these anchors are the current certificates of the [Certification
Certification Authority
Roiz](https://www.gov.br/iti/pt-br/assuntos/repositorio/repositorio-ac-raiz)
(Root CA). This set of anchors can be extended so that the Verifier
Verifier to support signatures from other CCIs.

This configuration is done in the Verifier's `web.xml` file. This file
file is inside the `WEB-INF` folder at the location where the application server
unzips the WAR file for execution. In the case of Apache Tomcat, the path
is `$CATALINA_BASE/webapps/verifier-$VERSION/WEB-INF/web.xml`.

In the `web.xml` file, two parameters are defined that relate to the
anchors used during the execution of the Verifier:

* the `trustAnchorsDirectory` parameter indicates in which directory the anchors will be
  the anchors will be fetched from. The operating system user running the
  Verifier application must have permission to read and write files in this
  directory;
* the `trustAnchorsURLs` parameter is given a comma-separated list of URLs
  comma separated URLs from which the certificates will be retrieved and saved into the
  directory specified above.

To add new trust anchors, simply gather them in the directory
directory specified by `trustAnchorsDirectory`, or add the desired URLs to the
web.xml` file. On initialization of the Verifier, it will read the
the files in the defined directory, and also download the certificates using the
the URLs listed. After modifications to the `web.xml` file, the
server needs to be restarted for the changes to take effect.

By default, the directory specified
is `/tmp/compliance-checker/Cache/trust-anchors/` and the URLs are from the
URLs of the current ICP-Brasil Root Certificate Authorities. The two ways of
obtained certificates can be used together or just one of them, depending
of them, depending on what suits the situation best. The directory specified
directory may not contain any certificates and all are obtained by downloading them
from the URLs, or if a certificate is not available from the URL
it can be added to the directory manually. More than one PCI can be
accepted at the same time in one instance of the Verifier.

##### Can the Verifier be used programmatically?

The Conformance Checker exposes the `/startup`, `/webreport`, and `/report` endpoints for submission.
and `/report` for submitting signatures via POST requests.

* The `/start` _endpoint returns an initial JSON evaluation of the submitted
  file, identifying whether it is verifiable by the application;
* the _endpoint_ `/webreport` is used by the Verifier's web platform to
  obtain the HTML signature reports;
* the _endpoint_ `/report` can be used to obtain reports for
  signature verification reports in XML or JSON programmatically.

Details of the usage and responses of the _endpoints_ are in the
[`IndexServlet.java`](reference-codes-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/IndexServlet.java) class
for `/startup`
and [`SimpleServlet.java`](reference-code-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/SimpleServlet.java)
to `/report`.

It is possible to restrict access from _endpoint_ `/report` by
configuration of the `web.xml` file, as described in the
[`SimpleServlet.java`] class (reference-codes-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/SimpleServlet.java). Additionally,
it is possible to make a configuration to limit requests to the _endpoint_
`/webreport`, detailed in
[`CompleteServlet.java`](reference-codes-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/CompleteServlet.java). Translated with www.DeepL.com/Translator (free version)


##### O que s??o os C??digos de Refer??ncia?

Os C??digos de Refer??ncia do Padr??o Brasileiro de Assinatura Digital constituem
a implementa????o de refer??ncia dos padr??es pertinentes a regulamenta????o de
assinaturas digitais no ??mbito da Infraestrutura de Chaves P??blicas Brasileira
(ICP-Brasil). T??m como p??blico-alvo fornecedores de aplica????es e plataformas
que desejam oferecer suporte a assinaturas digitais ICP-Brasil. Os C??digos de
Refer??ncia visam promover uma maior interoperabilidade entre tais aplica????es
e facilitar a implementa????o dos padr??es, simultaneamente oferecendo meios de
aprimorar o pr??prio conjunto normativo.

##### Qual ?? sua fun????o?

Criar e verificar assinaturas com certificado ICP-Brasil conforme o documento
[DOC-ICP-15](https://www.gov.br/iti/pt-br/centrais-de-conteudo/doc-icp-15-v-3-0-visao-geral-sobre-assin-dig-na-icp-brasil-pdf),
regulamentado pelo [Instituto Nacional de Tecnologia da
Informa????o](https://gov.br/iti) (ITI).

##### Como est??o organizados?

A principal caracter??stica que permeia os C??digos de Refer??ncia ?? o forte uso
de reflex??o, um conceito relacionado a metaprograma????o, para elaborar uma
orienta????o a componentes, permitindo a modulariza????o de algumas partes da base
de c??digo. Mais detalhes sobre esta abordagem podem ser lidos [nesta
monografia](https://repositorio.ufsc.br/handle/123456789/184160).

A base de c??digo ?? programada em Java e segue o padr??o
[Maven](https://maven.apache.org), portanto o c??digo-fonte est?? em
`codigos-de-referencia-core/src/main/java`. Testes unit??rios, localizados em
`codigos-de-referencia-core/src/test/java`, podem ser executados com `mvn
test`. Comandos `mvn` s??o usualmente executados na pasta raiz do c??digo-fonte.

A execu????o do comando `mvn package` gera automaticamente a documenta????o da base
de c??digo, localizada na pasta `docs`, e dois entreg??veis na forma de arquivos
WAR (_web application archive_), ambos na pasta `target`:

* O [Verificador de Conformidade](https://verificador.iti.gov.br), verificador
  oficial de assinaturas digitais da ICP-Brasil, cuja execu????o ?? iniciada
  a partir da classe `IndexServlet`. Como este sistema ?? utilizado em produ????o,
  sua documenta????o ?? detalhada abaixo.
* O [Assinador de Refer??ncia](https://pbad.labsec.ufsc.br/signer-hom), cuja
  execu????o ?? iniciada a partir da classe `SignerServlet`.

##### Como instalar e executar o Verificador de Conformidade?

Para executar corretamente, o Verificador necessita de no m??nimo 256 MB de
mem??ria RAM e um microprocessador de arquitetura x86-64 com dois n??cleos
f??sicos de processamento. Entretanto, o desempenho do Verificador ser??
diretamente proporcional aos recursos dispon??veis na m??quina, especialmente ao
verificar m??ltiplos arquivos simultaneamente. Portanto, ?? recomendado **8 GB de
mem??ria RAM e oito n??cleos f??sicos de processamento** para um desempenho
aceit??vel.

O pacote do Verificador ocupa cerca de **35 MB** em disco. Al??m disso, arquivos
de certificados digitais e listas de certificados revogados (LCRs) s??o
armazenados localmente para evitar transfer??ncias de arquivos repetidas,
e assim avaliar assinaturas digitais mais rapidamente. Neste contexto, a pasta
escolhida ?? `/tmp/verificador-de-conformidade`, e sugere-se que cerca de **1 GB
de espa??o em disco** seja reservado para o uso estendido desta funcionalidade.

Os C??digos de Refer??ncia necessitam de **Java 15** (OpenJDK, Eclipse J9 etc.)
rodando sob qualquer distribui????o GNU/Linux (Debian, openSUSE etc.) para que
sejam executados corretamente. Como o Verificador ?? uma aplica????o web, um
servidor de aplica????o que implemente o padr??o **Jakarta Servlet 5.0** (Apache
Tomcat 10.0+, GlassFish 6+) tamb??m ?? necess??rio.

O [servidor de homologa????o](https://pbad.labsec.ufsc.br/verifier-hom) do
Verificador utiliza **Ubuntu 20.04 LTS, OpenJDK 15 e [Apache Tomcat
10](https://tomcat.apache.org)**, por exemplo. Entretanto, ?? poss??vel executar
o Verificador de outras maneiras. O comando abaixo utiliza a plataforma
[Docker](https://docker.com) para executar uma inst??ncia local do Verificador
de Conformidade, compilada para o caminho `/path/to/verifier.war`,
disponibilizando-a em http://localhost:8080/verifier-docker.

```
docker run -it -p 8080:8080 tomcat:10.0.5-jdk15-adoptopenjdk-openj9 \
  -v /path/to/verifier.war:/usr/local/tomcat/webapps/verifier-docker.war
```

As depend??ncias do Verificador s??o obtidas automaticamente no seu processo de
empacotamento, e podem ser listadas explicitamente com o comando `mvn
dependency:tree`, ou verificadas nos arquivos `pom.xml`. O Verificador aceita
arquivos assinados digitalmente como sua entrada, sejam estes com assinaturas
anexadas, destacadas ou embarcadas. Como sa??da, produz relat??rios em PDF ou
HTML contendo v??rias informa????es sobre a validade das mesmas.

Para instalar o Verificador, o processo ?? dependente do servidor de aplica????o
escolhido. No caso do Apache Tomcat, basta copiar o arquivo WAR para a pasta de
aplica????es web do servidor de aplica????o (`$CATALINA_BASE/webapps`, onde
`$CATALINA_BASE` depende do [m??todo de
instala????o](https://tomcat.apache.org/tomcat-10.0-doc/introduction.html) do
Apache Tomcat).

Informa????es sobre o processo de verifica????o de assinaturas s??o registradas
atrav??s do servidor de aplica????o, por exemplo em `$CATALINA_BASE/logs` no caso
do Apache Tomcat. Problemas no funcionamento do Verificador ser??o expostos
atrav??s desses registros, que podem ser enviados para os desenvolvedores dos
C??digos de Refer??ncia pois auxiliam em poss??veis corre????es na base de c??digo.

O Verificador necessita realizar download de artefatos utilizados no processo
de verifica????o de assinaturas digitais atrav??s dos protocolos HTTP, HTTPS
e LDAP. Portanto, a comunica????o atrav??s destes protocolos pelo Verificador n??o
deve ser proibida pelo sistema operacional ou outras aplica????es.

##### O Verificador pode verificar assinaturas de outras ICPs?

O Verificador de Conformidade verifica assinaturas de acordo com um conjunto de
??ncoras de confian??a, utilizadas na checagem do caminho de certifica????o do
assinante atrav??s da extens??o de certificado [_Authority Information
Access_](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.2.1). No
??mbito da ICP-Brasil, estas ??ncoras s??o os certificados vigentes da [Autoridade
Certificadora
Raiz](https://www.gov.br/iti/pt-br/assuntos/repositorio/repositorio-ac-raiz)
(AC-Raiz). Este conjunto de ??ncoras pode ser estendido para que o Verificador
suporte assinaturas de outras ICPs.

Esta configura????o ?? feita no arquivo `web.xml` do Verificador. Este arquivo
est?? dentro da pasta `WEB-INF` no local onde o servidor de aplica????o
descompacta o arquivo WAR para execu????o. No caso do Apache Tomcat, o caminho
?? `$CATALINA_BASE/webapps/verifier-$VERSION/WEB-INF/web.xml`.

No arquivo `web.xml`, est??o definidos dois par??metros relacionados ??s ??ncoras
de confian??a utilizados durante a execu????o do Verificador:

* o par??metro `trustAnchorsDirectory` indica em qual diret??rio as ??ncoras ser??o
  buscadas. O usu??rio do sistema operacional executando a aplica????o do
  Verificador deve ter permiss??o de leitura e escrita de arquivos neste
  diret??rio;
* o par??metro `trustAnchorsURLs` recebe uma lista de URLs separadas por
  v??rgulas, atrav??s das quais os certificados ser??o obtidos e salvos no
  diret??rio especificado acima.

Para adicionar novas ??ncoras de confian??a, basta reuni-las no diret??rio
especificado por `trustAnchorsDirectory`, ou adicionar as URLs desejadas no
arquivo `web.xml`. Na inicializa????o do Verificador, ser?? feita a leitura dos
arquivos no diret??rio definido e tamb??m o download dos certificados utilizando
as URLs listadas. Ap??s modifica????es no arquivo `web.xml`, o servidor de
aplica????o precisa ser reiniciado para que as modifica????es entrem em vigor.

Por padr??o, o diret??rio especificado
?? `/tmp/verificador-de-conformidade/Cache/trust-anchors/` e as URLs s??o das
Autoridades Certificadoras Raiz vigentes da ICP-Brasil. Os dois modos de
obten????o dos certificados podem ser usados em conjunto ou ent??o apenas um
deles, dependendo do que mais se adequar na situa????o. O diret??rio especificado
pode n??o conter nenhum certificado e todos s??o obtidos atrav??s do download
pelas URLs, ou ent??o caso um certificado n??o esteja dispon??vel atrav??s de URL,
o mesmo pode ser adicionado ao diret??rio manualmente. Mais de uma ICP pode ser
aceita ao mesmo tempo em uma inst??ncia do Verificador.

##### O Verificador pode ser utilizado de maneira program??tica?

O Verificador de Conformidade exp??e os _endpoints_ `/inicio`, `/webreport`
e `/report` para submiss??o de assinaturas atrav??s de requisi????es POST.

* O _endpoint_ `/inicio` retorna uma avalia????o inicial em JSON do arquivo
  enviado, identificando se o mesmo ?? pass??vel de verifica????o pela aplica????o;
* o _endpoint_ `/webreport` ?? utilizado pela plataforma web do Verificador para
  obter os relat??rios de assinaturas em HTML;
* o _endpoint_ `/report` pode ser utilizado para obter relat??rios de
  verifica????o de assinaturas em XML ou JSON de maneira program??tica.

Detalhes do uso e respostas dos _endpoints_ est??o na classe
[`IndexServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/IndexServlet.java)
para `/inicio`
e [`SimpleServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/SimpleServlet.java)
para `/report`.

H?? a possibilidade de restri????o de acesso do _endpoint_ `/report` atrav??s da
configura????o do arquivo `web.xml`, como descrito na classe
[`SimpleServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/SimpleServlet.java). Adicionalmente,
?? poss??vel fazer uma configura????o de limita????o de requisi????es ao _endpoint_
`/webreport`, detalhada em
[`CompleteServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/CompleteServlet.java).
