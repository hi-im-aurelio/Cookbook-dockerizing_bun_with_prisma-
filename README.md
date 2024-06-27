# Dockerizing Bun with Prisma

This README provides a detailed recipe for setting up a Docker environment that uses Bun and Prisma. This configuration is useful to overcome known issues when running `bun` and `prisma` inside Docker containers.

## Known issues

Running Bun with Prisma in Docker containers can be challenging due to Prisma-specific dependencies on the Node API. Known issues include:

- [Bun doesn't run prisma generate or prisma migrate inside docker containers or WSL](https://github.com/oven-sh/bun/issues/5320)
- [bunx prisma generate breaking](https://github.com/prisma/prisma/issues/21277)

These issues can result in errors when running commands like `prisma generate` or `prisma migrate` in Docker containers. Or even the prism engine which depends on the Node.js API.

This cookbook was written with a lot of anger, I hope it makes you happy.

## Solution

Below is the Dockerfile recipe for setting up a container that uses Bun and Prisma without any problems.

### Dockerfile

```Dockerfile
# Use the official Bun image | Use a imagem oficial do Bun
FROM oven/bun:1-debian

WORKDIR /app

# Update apt | Atualizar o apt
RUN apt-get update

# Install curl using apt | Instalar o curl usando o apt
RUN apt-get install -y curl

# Add NodeSource repository and install Node.js | Adicionar repositório NodeSource e instalar Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
  apt-get install -y nodejs

# Set environment variable | Definir variável de ambiente
ENV NODE_ENV=development

# Copy package.json and bun.lockb | Copiar package.json e bun.lockb
COPY package.json bun.lockb /app/

# Copy prisma directory before installing dependencies | Copiar o diretório prisma antes de instalar as dependências
COPY prisma /app/prisma

# Install dependencies and generate Prisma client | Instalar dependências e gerar cliente Prisma
RUN set -e && \
  bun install && \
  bun run generate

# Copy the rest of the application | Copiar o restante da aplicação
COPY . /app/

# Expose port 3002 | Expor a porta 3002
EXPOSE 3002

# Define entrypoint | Definir entrypoint
ENTRYPOINT ["bun", "run", "dev"]
```

### Project Structure

Ensure that your project structure is organized as follows:

```
/app
  ├── prisma/
  │   └── schema.prisma
  ├── package.json
  ├── bun.lockb
  ├── Dockerfile
  ├── ...
```

### package.json

Make sure your `package.json` contains the necessary script to generate the Prisma client:

```json
{
  "scripts": {
    "generate": "bun prisma generate --schema=prisma/schema.prisma"
  }
}
```

### Docker commands

To build and run your container, use the following commands:

```sh
# Build the Docker image | Construir a imagem Docker
docker-compose -f docker-compose.dev.yml build --no-cache

# Run the Docker container | Rodar o contêiner Docker
docker-compose -f docker-compose.dev.yml up
```

With this setup, you should be able to build and run your Bun project with Prisma inside a Docker container without encountering the known issues mentioned above.
