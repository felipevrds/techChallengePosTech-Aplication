# Homologacao TC3 - preparacao local

Base: fbc299a2837c2146bc90a1e6a04d079c53f277e5. Dockerfile corrigido para OpenSSL 3.
Nenhum deploy ou publicacao foi executado nesta preparacao.

## Antes de implantar (exige autorizacao separada)

1. Publicar a imagem corrigida linux/amd64 no registry do fork e substituir REPLACE_WITH_PUBLISHED_SHA_TAG em kustomization.yaml por tag imutavel. Preferir digest verificado; Kustomize aceita o campo digest no lugar de newTag. A imagem atual e apenas local.
2. Se o pacote GHCR for privado, preparar imagePullSecret no namespace e adicionar sua referencia ao Deployment. Nenhuma credencial deve entrar no Git.
3. Preparar banco exclusivo de homologacao, schema, TLS e conectividade com a equipe de banco. O startup sobrescrito executa somente node dist/main.js. Nao ha db push, seed, migrations, initContainer ou Job automaticos.
4. Provisionar oficina-secret por canal seguro no namespace oficina-tc3. secret.yaml.example e ilustrativo, nao integra o Kustomize e nao deve receber valores reais.
5. DATABASE_URL e JWT_SECRET sao obrigatorios para este ambiente. PORT=3000 e NODE_ENV=production estao no ConfigMap. Combinar contrato JWT com Lambda/Gateway.
6. Para notificacoes reais, definir SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASS, SMTP_FROM e APP_URL conforme necessidade; credenciais ficam em Secret. Sem credenciais, o codigo tenta criar conta externa Ethereal ao enviar email.
7. Definir OTEL_EXPORTER_OTLP_ENDPOINT com URL de traces completa e acessivel (incluindo /v1/traces). O Alloy atual ainda nao teve recepcao OTLP de traces validada. Nao assumir localhost nem envio autenticado direto ao Grafana.
8. Medir capacidade real do Worker antes do deploy: requests por replica 100m/256Mi, limites 500m/512Mi, HPA 1-2; rollout pode exigir um terceiro pod. O control plane nao recebe toleration e nao e capacidade adicional presumida.

## Comportamento

Service somente ClusterIP: nenhuma exposicao publica, Ingress, NodePort ou LoadBalancer.
Probes TCP: detectam porta aberta, nao testam saude do banco nem regras de negocio.
Uma replica inicial; HPA pode ajustar depois. Ao automatizar deploys, preservar replicas geridas pelo HPA e evitar reaplicar replicas=1 indiscriminadamente.
UID/GID 100:101 correspondem ao appuser da imagem testada; revalidar se a base mudar.
Filesystem somente leitura, /tmp temporario, token Kubernetes desabilitado e capabilities removidas.
O handler SIGTERM da aplicacao encerra o processo apos tracing.shutdown; drenagem HTTP graciosa nao foi comprovada.

## Validacao local sem cluster

Renderizar somente este diretorio com Kustomize, nunca aplicar recursivamente k8s/.
Os manifests antigos k8s/postgres e k8s/secret.yaml nao fazem parte desta configuracao e nao devem ser utilizados neste fluxo.
Validar o resultado com kubeconform strict para Kubernetes 1.34.0.
A renderizacao com placeholder e valida estaticamente, mas NAO esta pronta para deploy ate substituir a imagem e cadastrar Secret.

## Pendencias

Publicacao GHCR autorizada, credenciais de pull se necessario, banco/schema aprovados, Secret seguro, medidas de capacidade, JWT/Gateway, traces, deploy automatico limitado ao namespace e revisao separada das vulnerabilidades npm.
