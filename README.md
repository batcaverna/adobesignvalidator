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


##### O que são os Códigos de Referência?

Os Códigos de Referência do Padrão Brasileiro de Assinatura Digital constituem
a implementação de referência dos padrões pertinentes a regulamentação de
assinaturas digitais no âmbito da Infraestrutura de Chaves Públicas Brasileira
(ICP-Brasil). Têm como público-alvo fornecedores de aplicações e plataformas
que desejam oferecer suporte a assinaturas digitais ICP-Brasil. Os Códigos de
Referência visam promover uma maior interoperabilidade entre tais aplicações
e facilitar a implementação dos padrões, simultaneamente oferecendo meios de
aprimorar o próprio conjunto normativo.

##### Qual é sua função?

Criar e verificar assinaturas com certificado ICP-Brasil conforme o documento
[DOC-ICP-15](https://www.gov.br/iti/pt-br/centrais-de-conteudo/doc-icp-15-v-3-0-visao-geral-sobre-assin-dig-na-icp-brasil-pdf),
regulamentado pelo [Instituto Nacional de Tecnologia da
Informação](https://gov.br/iti) (ITI).

##### Como estão organizados?

A principal característica que permeia os Códigos de Referência é o forte uso
de reflexão, um conceito relacionado a metaprogramação, para elaborar uma
orientação a componentes, permitindo a modularização de algumas partes da base
de código. Mais detalhes sobre esta abordagem podem ser lidos [nesta
monografia](https://repositorio.ufsc.br/handle/123456789/184160).

A base de código é programada em Java e segue o padrão
[Maven](https://maven.apache.org), portanto o código-fonte está em
`codigos-de-referencia-core/src/main/java`. Testes unitários, localizados em
`codigos-de-referencia-core/src/test/java`, podem ser executados com `mvn
test`. Comandos `mvn` são usualmente executados na pasta raiz do código-fonte.

A execução do comando `mvn package` gera automaticamente a documentação da base
de código, localizada na pasta `docs`, e dois entregáveis na forma de arquivos
WAR (_web application archive_), ambos na pasta `target`:

* O [Verificador de Conformidade](https://verificador.iti.gov.br), verificador
  oficial de assinaturas digitais da ICP-Brasil, cuja execução é iniciada
  a partir da classe `IndexServlet`. Como este sistema é utilizado em produção,
  sua documentação é detalhada abaixo.
* O [Assinador de Referência](https://pbad.labsec.ufsc.br/signer-hom), cuja
  execução é iniciada a partir da classe `SignerServlet`.

##### Como instalar e executar o Verificador de Conformidade?

Para executar corretamente, o Verificador necessita de no mínimo 256 MB de
memória RAM e um microprocessador de arquitetura x86-64 com dois núcleos
físicos de processamento. Entretanto, o desempenho do Verificador será
diretamente proporcional aos recursos disponíveis na máquina, especialmente ao
verificar múltiplos arquivos simultaneamente. Portanto, é recomendado **8 GB de
memória RAM e oito núcleos físicos de processamento** para um desempenho
aceitável.

O pacote do Verificador ocupa cerca de **35 MB** em disco. Além disso, arquivos
de certificados digitais e listas de certificados revogados (LCRs) são
armazenados localmente para evitar transferências de arquivos repetidas,
e assim avaliar assinaturas digitais mais rapidamente. Neste contexto, a pasta
escolhida é `/tmp/verificador-de-conformidade`, e sugere-se que cerca de **1 GB
de espaço em disco** seja reservado para o uso estendido desta funcionalidade.

Os Códigos de Referência necessitam de **Java 15** (OpenJDK, Eclipse J9 etc.)
rodando sob qualquer distribuição GNU/Linux (Debian, openSUSE etc.) para que
sejam executados corretamente. Como o Verificador é uma aplicação web, um
servidor de aplicação que implemente o padrão **Jakarta Servlet 5.0** (Apache
Tomcat 10.0+, GlassFish 6+) também é necessário.

O [servidor de homologação](https://pbad.labsec.ufsc.br/verifier-hom) do
Verificador utiliza **Ubuntu 20.04 LTS, OpenJDK 15 e [Apache Tomcat
10](https://tomcat.apache.org)**, por exemplo. Entretanto, é possível executar
o Verificador de outras maneiras. O comando abaixo utiliza a plataforma
[Docker](https://docker.com) para executar uma instância local do Verificador
de Conformidade, compilada para o caminho `/path/to/verifier.war`,
disponibilizando-a em http://localhost:8080/verifier-docker.

```
docker run -it -p 8080:8080 tomcat:10.0.5-jdk15-adoptopenjdk-openj9 \
  -v /path/to/verifier.war:/usr/local/tomcat/webapps/verifier-docker.war
```

As dependências do Verificador são obtidas automaticamente no seu processo de
empacotamento, e podem ser listadas explicitamente com o comando `mvn
dependency:tree`, ou verificadas nos arquivos `pom.xml`. O Verificador aceita
arquivos assinados digitalmente como sua entrada, sejam estes com assinaturas
anexadas, destacadas ou embarcadas. Como saída, produz relatórios em PDF ou
HTML contendo várias informações sobre a validade das mesmas.

Para instalar o Verificador, o processo é dependente do servidor de aplicação
escolhido. No caso do Apache Tomcat, basta copiar o arquivo WAR para a pasta de
aplicações web do servidor de aplicação (`$CATALINA_BASE/webapps`, onde
`$CATALINA_BASE` depende do [método de
instalação](https://tomcat.apache.org/tomcat-10.0-doc/introduction.html) do
Apache Tomcat).

Informações sobre o processo de verificação de assinaturas são registradas
através do servidor de aplicação, por exemplo em `$CATALINA_BASE/logs` no caso
do Apache Tomcat. Problemas no funcionamento do Verificador serão expostos
através desses registros, que podem ser enviados para os desenvolvedores dos
Códigos de Referência pois auxiliam em possíveis correções na base de código.

O Verificador necessita realizar download de artefatos utilizados no processo
de verificação de assinaturas digitais através dos protocolos HTTP, HTTPS
e LDAP. Portanto, a comunicação através destes protocolos pelo Verificador não
deve ser proibida pelo sistema operacional ou outras aplicações.

##### O Verificador pode verificar assinaturas de outras ICPs?

O Verificador de Conformidade verifica assinaturas de acordo com um conjunto de
âncoras de confiança, utilizadas na checagem do caminho de certificação do
assinante através da extensão de certificado [_Authority Information
Access_](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.2.1). No
âmbito da ICP-Brasil, estas âncoras são os certificados vigentes da [Autoridade
Certificadora
Raiz](https://www.gov.br/iti/pt-br/assuntos/repositorio/repositorio-ac-raiz)
(AC-Raiz). Este conjunto de âncoras pode ser estendido para que o Verificador
suporte assinaturas de outras ICPs.

Esta configuração é feita no arquivo `web.xml` do Verificador. Este arquivo
está dentro da pasta `WEB-INF` no local onde o servidor de aplicação
descompacta o arquivo WAR para execução. No caso do Apache Tomcat, o caminho
é `$CATALINA_BASE/webapps/verifier-$VERSION/WEB-INF/web.xml`.

No arquivo `web.xml`, estão definidos dois parâmetros relacionados às âncoras
de confiança utilizados durante a execução do Verificador:

* o parâmetro `trustAnchorsDirectory` indica em qual diretório as âncoras serão
  buscadas. O usuário do sistema operacional executando a aplicação do
  Verificador deve ter permissão de leitura e escrita de arquivos neste
  diretório;
* o parâmetro `trustAnchorsURLs` recebe uma lista de URLs separadas por
  vírgulas, através das quais os certificados serão obtidos e salvos no
  diretório especificado acima.

Para adicionar novas âncoras de confiança, basta reuni-las no diretório
especificado por `trustAnchorsDirectory`, ou adicionar as URLs desejadas no
arquivo `web.xml`. Na inicialização do Verificador, será feita a leitura dos
arquivos no diretório definido e também o download dos certificados utilizando
as URLs listadas. Após modificações no arquivo `web.xml`, o servidor de
aplicação precisa ser reiniciado para que as modificações entrem em vigor.

Por padrão, o diretório especificado
é `/tmp/verificador-de-conformidade/Cache/trust-anchors/` e as URLs são das
Autoridades Certificadoras Raiz vigentes da ICP-Brasil. Os dois modos de
obtenção dos certificados podem ser usados em conjunto ou então apenas um
deles, dependendo do que mais se adequar na situação. O diretório especificado
pode não conter nenhum certificado e todos são obtidos através do download
pelas URLs, ou então caso um certificado não esteja disponível através de URL,
o mesmo pode ser adicionado ao diretório manualmente. Mais de uma ICP pode ser
aceita ao mesmo tempo em uma instância do Verificador.

##### O Verificador pode ser utilizado de maneira programática?

O Verificador de Conformidade expõe os _endpoints_ `/inicio`, `/webreport`
e `/report` para submissão de assinaturas através de requisições POST.

* O _endpoint_ `/inicio` retorna uma avaliação inicial em JSON do arquivo
  enviado, identificando se o mesmo é passível de verificação pela aplicação;
* o _endpoint_ `/webreport` é utilizado pela plataforma web do Verificador para
  obter os relatórios de assinaturas em HTML;
* o _endpoint_ `/report` pode ser utilizado para obter relatórios de
  verificação de assinaturas em XML ou JSON de maneira programática.

Detalhes do uso e respostas dos _endpoints_ estão na classe
[`IndexServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/IndexServlet.java)
para `/inicio`
e [`SimpleServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/SimpleServlet.java)
para `/report`.

Há a possibilidade de restrição de acesso do _endpoint_ `/report` através da
configuração do arquivo `web.xml`, como descrito na classe
[`SimpleServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/SimpleServlet.java). Adicionalmente,
é possível fazer uma configuração de limitação de requisições ao _endpoint_
`/webreport`, detalhada em
[`CompleteServlet.java`](codigos-de-referencia-core/src/main/java/br/ufsc/labsec/signature/conformanceVerifier/CompleteServlet.java).
