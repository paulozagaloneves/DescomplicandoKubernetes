# Aula ao Vivo

## Criando imagem standard

### Criando imagem

**Arquivo:** dockerfiles/Dockerfile.standard

```dockerfile
FROM golang:1.25

WORKDIR /app

COPY src/go.mod .
COPY src/main.go ./


RUN go build -o main .

CMD ["./main"]
```

```bash
$ docker image build -t cash-flow:1.0 -f dockerfiles/Dockerfile.standard .
$
$ docker image ls                                                                        
                                                                        i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA 
linuxtips/cash-flow:1.0                     280fa7eb9ed8        937MB             0B   
```

**Nota:** Temos uma imagem funcional porem muito grande

### Validando vulnerabilidades

**Docker scout**

**Docker Scout** é uma ferramenta da Docker que ajuda a analisar, monitorar e melhorar a segurança das imagens de containers. Ela verifica vulnerabilidades, sugere melhorias e mostra informações sobre dependências e práticas recomendadas.

**Principais funções:**

- Detecta vulnerabilidades em imagens Docker.
- Sugere atualizações de pacotes e dependências.
- Mostra o histórico de camadas da imagem.
- Ajuda a manter imagens mais seguras e eficientes.
- Você pode usar o Docker Scout via CLI ou integrado ao Docker Hub, facilitando a análise contínua das imagens durante o desenvolvimento e o deploy.

**Instalar**

**Referência:** https://docs.docker.com/scout/install/

```bash
$ curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh -o install-scout.sh
$ sh install-scout.sh
```

**Verificar vulnerabilidades**

```bash
$ docker scout quickview linuxtips/cash-flow:1.0
    ✓ SBOM of image already cached, 293 packages indexed

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target     │  linuxtips/cash-flow:1.0  │    0C     0H     5M    90L  
   digest   │  280fa7eb9ed8             │                             
 Base image │  golang:1                 │    0C     0H     5M    90L  

What's next:
    View vulnerabilities → docker scout cves linuxtips/cash-flow:1.0
    Include policy results in your quickview by supplying an organization → docker scout quickview linuxtips/cash-flow:1.0 --org <organization>

```

## Criando imagens Multistage

**O que é Multistage**

**Multistage** em Dockerfiles é uma técnica que permite usar múltiplas etapas (stages) de construção em um único Dockerfile. Isso é útil para criar imagens menores e mais seguras, pois você pode separar o processo de build (compilação, testes, etc.) do ambiente final de execução.

Como funciona:

- Você define várias etapas usando a instrução FROM várias vezes.
- Cada etapa pode usar uma imagem base diferente.
- Você copia apenas os artefatos necessários da etapa de build para a etapa final, descartando arquivos e dependências desnecessárias.

### Criando imagem

```dockerfile
# Build Stage
FROM golang:1.25 AS builder

WORKDIR /app

COPY src/go.mod .
COPY src/main.go ./

RUN CGO_ENABLED=0 GOOS=linux go build -o main .


# Final stage
FROM alpine:3.23.3

COPY --from=builder /app/main /

CMD ["/main"]
```

**Build da imagem**

```bash
$ docker image build -t linuxtips/cash-flow:4.0 -f ./dockerfiles/Dockerfile.multistage .
[+] Building 1.7s (15/15) FINISHED                                                                                                       docker:default
 => [internal] load build definition from Dockerfile.multistage                                                                                    0.0s
 => => transferring dockerfile: 276B                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/golang:1.25                                                                                     1.5s
 => [internal] load metadata for docker.io/library/alpine:3.23.3                                                                                   1.4s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                      0.0s
 => [auth] library/golang:pull token for registry-1.docker.io                                                                                      0.0s
 => [internal] load .dockerignore                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                    0.0s
 => [builder 1/5] FROM docker.io/library/golang:1.25@sha256:cc737435e2742bd6da3b7d575623968683609a3d2e0695f9d85bee84071c08e6                       0.0s
 => => resolve docker.io/library/golang:1.25@sha256:cc737435e2742bd6da3b7d575623968683609a3d2e0695f9d85bee84071c08e6                               0.0s
 => [internal] load build context                                                                                                                  0.0s
 => => transferring context: 85B                                                                                                                   0.0s
 => CACHED [stage-1 1/2] FROM docker.io/library/alpine:3.23.3@sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c3f659              0.0s
 => CACHED [builder 2/5] WORKDIR /app                                                                                                              0.0s
 => CACHED [builder 3/5] COPY src/go.mod .                                                                                                         0.0s
 => CACHED [builder 4/5] COPY src/main.go ./                                                                                                       0.0s
 => CACHED [builder 5/5] RUN CGO_ENABLED=0 GOOS=linux go build -o main .                                                                           0.0s
 => [stage-1 2/2] COPY --from=builder /app/main /                                                                                                  0.0s
 => exporting to image                                                                                                                             0.0s
 => => exporting layers                                                                                                                            0.0s
 => => writing image sha256:dece224e2524de6d4765fe36c5bf0295ab48d06383a20066187c61837f930d69                                                       0.0s
 => => naming to docker.io/linuxtips/cash-flow:4.0                                               
```

**Verificando tamanho da imagem**

```bash
$ docker image ls                                                                       
                                                                        i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
linuxtips/cash-flow:1.0                     280fa7eb9ed8        937MB             0B        
linuxtips/cash-flow:4.0                     dece224e2524       16.5MB             0B        
```

**Nota:** Continuamos com uma imagem (4.0) funcional porem bem menor (cerca de 1% do tamanho inicial)

### Validando vulnerabilidades

```bash
$ docker scout quickview linuxtips/cash-flow:4.0
    ✓ Image stored for indexing
    ✓ Indexed 22 packages

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target             │  linuxtips/cash-flow:4.0  │    0C     0H     1M     0L  
   digest           │  dece224e2524             │                             
 Base image         │  alpine:3                 │    0C     0H     1M     0L  
 Updated base image │  alpine:3.21              │    0C     0H     1M     0L  
                    │                           │                             

What's next:
    View vulnerabilities → docker scout cves linuxtips/cash-flow:4.0
    View base image update recommendations → docker scout recommendations linuxtips/cash-flow:4.0
    Include policy results in your quickview by supplying an organization → docker scout quickview linuxtips/cash-flow:4.0 --org <organization>

```

## Criando imagens seguras Multistage com Wolfi

**O que é a Chainguard e suas imagens Wolfi**

**Chainguard** é uma empresa focada em segurança de software supply chain, oferecendo soluções para garantir imagens de containers seguras, verificáveis e com mínimo de vulnerabilidades.

**Wolfi** é uma distribuição Linux minimalista criada pela Chainguard, projetada para containers. Suas imagens são:

- Sem pacotes desnecessários (reduz superfície de ataque)
- Sem usuários root por padrão
- Atualizadas frequentemente para evitar vulnerabilidades
- Compatíveis com OCI/Docker
- Imagens Wolfi são usadas como base para containers seguros, como o exemplo cgr.dev/chainguard/static:latest, garantindo builds mais confiáveis e auditáveis.

**Referências:**

- [https://github.com/wolfi-dev](https://github.com/wolfi-dev)
- [https://images.chainguard.dev/](https://images.chainguard.dev/)
- [Documentation](https://edu.chainguard.dev/)

```dockerfile
# Build Stage
FROM golang:1.25 AS builder

WORKDIR /app

COPY src/go.mod .
COPY src/main.go ./

RUN CGO_ENABLED=0 GOOS=linux go build -o main .


# Final stage
FROM cgr.dev/chainguard/static:latest

COPY --from=builder /app/main /

CMD ["/main"]
```

```bash
$ docker image build -t cash-flow:2.0 -f dockerfiles/Dockerfile.wolfi .
$
$ docker image ls                                                                       
                                                                        i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
linuxtips/cash-flow:1.0                     280fa7eb9ed8        937MB             0B        
linuxtips/cash-flow:2.0                     4779a976b9c3       10.2MB             0B     
linuxtips/cash-flow:4.0                     dece224e2524       16.5MB             0B        
$
```

**Nota:** Como é possível verificar a imagem (2.0) continuou a reduzir o tamanho.

### Validando vulnerabilidades

```bash
$ docker scout quickview linuxtips/cash-flow:2.0      
    ✓ SBOM of image already cached, 9 packages indexed

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target   │  linuxtips/cash-flow:2.0  │    0C     0H     0M     0L  
   digest │  4779a976b9c3             │                             

What's next:
    Include policy results in your quickview by supplying an organization → docker scout quickview linuxtips/cash-flow:2.0 --org <organization>

```

**Nota:** Como se pode verificar zeramos as vulnerabilidades da imagem.

## Criando imagens seguras Multistage distroless

**O que são imagens distroless**

Imagens **distroless** são imagens de container que não incluem um sistema operacional completo, como Alpine ou Debian. Elas contêm apenas o aplicativo e suas dependências essenciais, sem ferramentas de depuração, shells ou gerenciadores de pacotes.

**Principais características:**

- Menor tamanho de imagem.
- Superfície de ataque reduzida (mais seguras).
- Não permitem acesso via shell (ex: bash, sh).
- Ideais para produção, pois só carregam o necessário para rodar o app.

Exemplo de uso:

No lugar de usar FROM debian ou FROM alpine, você pode usar FROM gcr.io/distroless/base ou FROM gcr.io/distroless/python3, por exemplo.

Essas imagens são recomendadas quando você quer máxima segurança e simplicidade no ambiente de execução do seu container.

```dockerfile
# Build Stage
FROM golang:1.25 AS builder

WORKDIR /app

COPY src/go.mod .
COPY src/main.go ./

RUN CGO_ENABLED=0 GOOS=linux go build -o main .


# Final stage
FROM gcr.io/distroless/static-debian12

COPY --from=builder /app/main /

CMD ["/main"]
```

```bash
$ docker image build -t cash-flow:3.0 -f dockerfiles/Dockerfile.distroless .
$                                                                      
$ docker image ls                                                                       
                                                                        i Info →   U  In Use
IMAGE                                       ID             DISK USAGE   CONTENT SIZE   EXTRA
linuxtips/cash-flow:1.0                     280fa7eb9ed8        937MB             0B        
linuxtips/cash-flow:2.0                     4779a976b9c3       10.2MB             0B     
linuxtips/cash-flow:3.0                     516ea11b89b0       10.1MB             0B  
linuxtips/cash-flow:4.0                     dece224e2524       16.5MB             0B        
$
```

**Nota:** Como é possível verificar a imagem (3.0) continuou a reduzir o tamanho.

### Validando vulnerabilidades

```bash
$ docker scout quickview linuxtips/cash-flow:3.0
    ✓ SBOM of image already cached, 6 packages indexed

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

 Target     │  linuxtips/cash-flow:3.0            │    0C     0H     0M     0L  
   digest   │  516ea11b89b0                       │                             
 Base image │  distroless/static-debian12:latest  │    0C     0H     0M     0L  

What's next:
    Include policy results in your quickview by supplying an organization → docker scout quickview linuxtips/cash-flow:3.0 --org <organization>


```

**Nota:** Como se pode verificar, mesmo com uma imagem distroless (bem reduzida) ainda conseguimos zerar as vulnerabilidades da imagem.

## Docker history

O comando **docker history** exibe o histórico de camadas (layers) de uma imagem Docker. Ele mostra, em ordem, como a imagem foi construída, listando cada instrução do Dockerfile (como RUN, COPY, ADD) que gerou uma nova camada, junto com informações como tamanho, autor, data e comando usado.

**Exemplo de uso:**

```sh
docker history nome-da-imagem
```

Isso ajuda a entender como a imagem foi criada, identificar camadas grandes e otimizar o Dockerfile.

### Docker history das imagens criadas anteriormente

**Imagem standard**

```bash
$ docker history linuxtips/cash-flow:1.0        
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
280fa7eb9ed8   23 hours ago   CMD ["./main"]                                  0B        buildkit.dockerfile.v0
<missing>      23 hours ago   RUN /bin/sh -c go build -o main . # buildkit    89.5MB    buildkit.dockerfile.v0
<missing>      23 hours ago   COPY src/main.go ./ # buildkit                  446B      buildkit.dockerfile.v0
<missing>      23 hours ago   COPY src/go.mod . # buildkit                    23B       buildkit.dockerfile.v0
<missing>      39 hours ago   WORKDIR /app                                    0B        buildkit.dockerfile.v0
<missing>      3 days ago     WORKDIR /go                                     0B        buildkit.dockerfile.v0
<missing>      3 days ago     RUN /bin/sh -c mkdir -p "$GOPATH/src" "$GOPA…   0B        buildkit.dockerfile.v0
<missing>      3 days ago     COPY /target/ / # buildkit                      206MB     buildkit.dockerfile.v0
<missing>      3 days ago     ENV PATH=/go/bin:/usr/local/go/bin:/usr/loca…   0B        buildkit.dockerfile.v0
<missing>      3 days ago     ENV GOPATH=/go                                  0B        buildkit.dockerfile.v0
<missing>      3 days ago     ENV GOTOOLCHAIN=local                           0B        buildkit.dockerfile.v0
<missing>      3 days ago     ENV GOLANG_VERSION=1.25.7                       0B        buildkit.dockerfile.v0
<missing>      3 days ago     RUN /bin/sh -c set -eux;  apt-get update;  a…   276MB     buildkit.dockerfile.v0
<missing>      5 days ago     RUN /bin/sh -c set -eux;  apt-get update;  a…   185MB     buildkit.dockerfile.v0
<missing>      5 days ago     RUN /bin/sh -c set -eux;  apt-get update;  a…   60.2MB    buildkit.dockerfile.v0
<missing>      6 days ago     # debian.sh --arch 'amd64' out/ 'trixie' '@1…   120MB     debuerreotype 0.17
```

**Multistage com Alpine**

```bash
$ docker history linuxtips/cash-flow:4.0
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
dece224e2524   36 minutes ago   CMD ["/main"]                                   0B        buildkit.dockerfile.v0
<missing>      36 minutes ago   COPY /app/main / # buildkit                     8.05MB    buildkit.dockerfile.v0
<missing>      11 days ago      CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      11 days ago      ADD alpine-minirootfs-3.23.3-x86_64.tar.gz /…   8.44MB    buildkit.dockerfile.v0
```

**Multistage com Wolfi**

```bash
$ docker history linuxtips/cash-flow:2.0
IMAGE          CREATED        CREATED BY                    SIZE      COMMENT
4779a976b9c3   23 hours ago   CMD ["/main"]                 0B        buildkit.dockerfile.v0
<missing>      23 hours ago   COPY /app/main / # buildkit   8.05MB    buildkit.dockerfile.v0
<missing>      9 days ago     apko                          2.1MB     static by Chainguard
```

**Multistage com imagem distroless**

```bash
$ docker history linuxtips/cash-flow:3.0
IMAGE          CREATED        CREATED BY                    SIZE      COMMENT
516ea11b89b0   23 hours ago   CMD ["/main"]                 0B        buildkit.dockerfile.v0
<missing>      23 hours ago   COPY /app/main / # buildkit   8.05MB    buildkit.dockerfile.v0
<missing>      N/A                                          236kB     
<missing>      N/A                                          346B      
<missing>      N/A                                          497B      
<missing>      N/A                                          0B        
<missing>      N/A                                          64B       
<missing>      N/A                                          0B        
<missing>      N/A                                          149B      
<missing>      N/A                                          0B        
<missing>      N/A                                          82.1kB    
<missing>      N/A                                          1.47MB    
<missing>      N/A                                          22.9kB    
<missing>      N/A                                          271kB   
```