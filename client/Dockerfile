FROM node:16.15.1 as builder

ARG VITE_NODE_ENV
ARG VITE_BOT_SERVICE_URL
ARG VITE_CLIENT_PORT
ARG VITE_NODE_URL
ARG VITE_COMPILER_URL
ARG VITE_WS_URL
ARG VITE_FAUCET_PUBLIC_ADDRESS
ARG VITE_NETWORK_ID
ARG VITE_RESPONDER_HOST
ARG VITE_RESPONDER_PORT

RUN mkdir -p /home/node/app/client
WORKDIR /home/node/app/client
COPY ./client/package*.json /home/node/app/client/
RUN chown -R node:node /home/node/

USER node
COPY --chown=node:node ./contract /home/node/app/contract
COPY --chown=node:node ./client /home/node/app/client
RUN cd ../contract && npm install
RUN cd ../client && npm ci
RUN npm run build

FROM nginx:1.13.7-alpine
COPY ./client/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./client/nginx/default.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /home/node/app/client/dist /usr/share/nginx/html
