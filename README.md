# Dasboard-Kubernetes-com-AD
Kubernetes com Active Directory

Olá pessoal, esse tutorial talvez seja util para alguém que esteja configurando autenticação do dashboard-kubernetes com AD/LDAP, deixando claro que existem outras maneiras de ser feito a mesma coisa.

-Servidor - Ubuntu 20.04.2 LTS
-kube-apiserver v1.22.0

Partindo do pressuposto que já se tem o kubernetes instalado e também o Dashboard-kubernetes com serviço e porta funcionando vamos instalar o servidor de autenticação para o dashboard-kubernetes autenticar com usuários do AD.


1- Gerar as chaves que serão utilizadas pelo servidor de autenticação DEX, arquivo gencert.sh o mesmo esta em shell script mas deve ser 
customizado conforme sua preferência.
<pre>
OBS: verifique o IP do seu servidor ou dominio para colocar no script
./gencert.sh
</pre>


2- Copiar as chaves criadas para pasta "/etc/ssl/certs/"
<pre>
cp /etc/kubernetes/yaml/ssl/cert.pem /etc/ssl/certs/
cp /etc/kubernetes/yaml/ssl/ca.pem /etc/ssl/certs/
</pre>


3- Construir o container/Pod do DEX arquivo DEX.yaml, altere as informações conforme seu servidor.
<pre>
OBS: verifique o IP do seu servidor ou dominio para colocar no arquivo DEX.yaml assim como os dados de acesso do Active Directory
kubectl apply -f DEX.yaml
</pre>


4- Adicionar na API do Kubernetes os dados do DEX conforme exemplo abaixo:
<pre>
vim /etc/kubernetes/manifests/kube-apiserver.yaml
spec:
   containers:
   - command:
     - kube-apiserver
     - --oidc-issuer-url=https://DOMINIO_DO_SERVIDOR_OU_IP:32001
     - --oidc-client-id=kubernetes
     - --oidc-username-claim=email
     - --oidc-groups-claim=groups
</pre>


5- Construir o container/Pod do Oauth2-proxy, arquivo OAUTH2.yaml altere as informações conforme dados do seu servidor.
<pre>
OBS: Nessa configuração temos a autenticação com cookie que pode ser alterado o tempo de atualização e também o tempo de vida do mesmo, 
altere conforme sua preferência. Também é necessário colocar o endereço do serviço do dashboard-kubernetes.
</pre>


6- Implementar as regras de permissões dos usuários do grupo do AD, no nosso caso permissões a nível de cluster RBAC.
<pre>
OBS: edite o arquivo e coloque o nome do grupo do AD no arquivo RBAC.yaml que o usuário esta inserido.
kubectl apply -f RBAC.yaml
</pre>

7- Acessar seu navegador e colocar o endereço do Oauth2-proxy: 
<pre>
https://DOMINIO_DO_SERVIDOR_OU_IP:32045
-Inserir seus dados do AD, no caso seu email, exemplo: usuario@ad.empresa.br e senha.
</pre>
