# Usa a imagem oficial do Node.js como base
FROM node:14

# Define o diretório de trabalho dentro do container
WORKDIR /usr/src/app

# Copia os arquivos do backend para o diretório de trabalho
COPY . .

# Instala as dependências
RUN npm install

# Expõe a porta em que a aplicação será executada
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["node", "app.js"]
