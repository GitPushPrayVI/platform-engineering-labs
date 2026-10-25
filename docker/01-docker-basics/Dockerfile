
# Node Js as a base Img
FROM node:14
# Sets our working directory in /app
WORKDIR /app
# Copy the json into the working directory (/app)
COPY package.json .
# Install all the dependencies during image build
RUN npm install
# Copyies files from the build into /app (ignores .dockeringnore)
COPY . .
# Port the application on 3000
EXPOSE 3000
# Run Environment 
CMD [ "node", "app.mjs"]