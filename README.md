# UsingOpenSSL

1. Geração de Chaves RSA
O RSA é o padrão tradicional de criptografia baseada na dificuldade de fatorar números primos grandes.

openssl genrsa: Gera uma chave privada RSA.

Uso comum: openssl genrsa -out chave.pem 2048.

Nota: O tamanho da chave (ex: 2048) deve ser sempre o último argumento.

openssl rsa -in <arq> -text: Lê um arquivo de chave RSA e exibe seus componentes em formato legível (Modulus, expoentes, números primos, etc.).

Dica: Adicione -noout para não repetir a versão codificada em Base64 na tela.

2. Geração de Chaves de Curva Elíptica (ECC)
O ECC é o padrão moderno, oferecendo a mesma segurança que o RSA, mas com chaves muito menores e menos esforço do processador.

openssl ecparam: Utilizado para manipular parâmetros de curvas elípticas.

-list_curves: Lista todas as curvas suportadas pelo seu sistema.

-genparam: Gera um arquivo de parâmetros baseado em uma curva específica (ex: secp384r1).

openssl genpkey: Um comando mais moderno e versátil para gerar chaves privadas.

Pode ser usado com -paramfile (usando o arquivo gerado pelo ecparam) ou definindo o algoritmo e a curva diretamente com -algorithm EC.

openssl ec -in <arq> -text: Semelhante ao comando RSA, mas específico para inspecionar chaves de Curva Elíptica.

3. Comandos Auxiliares e Conceitos
-out <nome_do_arquivo>: Redireciona a saída do comando para um arquivo em vez de apenas exibir no terminal.

-in <nome_do_arquivo>: Define o arquivo de entrada que o OpenSSL deve ler/processar.

.pem: Não é um comando, mas a extensão de arquivo padrão discutida. Significa Privacy Enhanced Mail e indica que o conteúdo está codificado em Base64 (texto ASCII), facilitando o copiar/colar.


# PART 2 #
1. Comandos de Geração (openssl req)
O utilitário req é o "canivete suíço" para criar identidades digitais. A principal diferença está na flag utilizada:

openssl req -x509: Gera um Certificado Autoassinado (Self-Signed).

Para que serve: Cria um certificado final que você mesmo assina. É útil para testes internos, mas navegadores exibirão um alerta de segurança porque não há uma Autoridade Certificadora (CA) confiável validando-o.

openssl req -new: Gera uma CSR (Certificate Signing Request).

Para que serve: Cria um arquivo de solicitação que contém sua chave pública e seus dados. Você envia este arquivo para uma CA (como Let's Encrypt ou DigiCert) para que eles emitam um certificado assinado e confiável.

-key <arquivo.pem>: Indica qual chave privada será usada para gerar o certificado ou a CSR.

-subj "/C=BR/ST=RJ/CN=meusite.com": Atalho para preencher os dados de identidade (País, Estado, Nome Comum) diretamente no comando, evitando as perguntas interativas do terminal.

2. Comandos de Inspeção
Como os arquivos PEM são apenas códigos em Base64, você precisa destes comandos para ler o que está dentro deles:

openssl x509 -in cert.pem -noout -text: Exibe os detalhes de um Certificado.

O que verificar: Data de validade, quem emitiu (Issuer), o nome do dono (Subject) e o campo SAN (Subject Alternative Name).

openssl req -in request.csr -noout -text: Exibe os detalhes de uma CSR.

O que verificar: Garante que as informações enviadas para a CA estão corretas antes de você pagar ou solicitar a emissão.

3. Conceitos Chave de Identidade
Ao rodar os comandos acima, você lida com campos críticos para a confiança do navegador:

Common Name (CN): O endereço principal do site (ex: google.com). Antigamente era o único campo validado.

Subject Alternative Name (SAN): O padrão moderno. Permite que um único certificado proteja vários domínios ou subdomínios (ex: blog.site.com, loja.site.com).

Distinguished Name (DN): O conjunto de informações (País, Organização, etc.) que identifica a entidade.

# PART 3 Troubleshooting: #

1. Comandos para Chaves RSA (O Método do Modulus)
No RSA, o componente único que liga o par é o Modulus. Se o Modulus do certificado e da chave forem iguais, eles combinam.

openssl x509 -noout -modulus -in cert.pem: Extrai o modulus do certificado.

openssl rsa -noout -modulus -in chave.key: Extrai o modulus da chave privada.

O "Pulo do Gato": Usando Hashes
Como o modulus é um código hexadecimal gigantesco e difícil de comparar visualmente, usamos um hash (como MD5) para gerar uma "impressão digital" curta:

openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in chave.key | openssl md5
Se o resultado for igual (ex: a1b2c3d4...), o par está correto.

2. Comandos para Chaves de Curva Elíptica (ECC)
Chaves ECC não possuem um "modulus". Para elas, comparamos a chave pública inteira.

openssl x509 -in cert.pem -noout -pubkey: Extrai a chave pública do certificado.

openssl ec -in chave.key -pubout: Extrai a chave pública do arquivo de chave privada.

Nota: Assim como no RSA, você pode passar o resultado para o openssl md5 para facilitar a comparação.

3. O Comando Moderno: openssl pkey
A partir da versão 1.1 do OpenSSL, o utilitário pkey foi introduzido para ser genérico. Ele detecta automaticamente se a chave é RSA ou ECC.

openssl pkey -in chave.key -text -noout: Inspeciona a chave (independente do tipo).

openssl pkey -in chave.key -pubout: Extrai a chave pública (funciona para RSA e ECC).

#PART 4 Validando certificados:#

1. Comandos de Conexão e Extração
O comando principal aqui é o s_client, que abre uma conexão SSL/TLS com um servidor remoto.

openssl s_client -connect <site>:443: Conecta ao servidor e exibe o certificado em formato Base64 (PEM).

Uso: Você copia o bloco entre -----BEGIN CERTIFICATE----- e -----END CERTIFICATE----- para um arquivo local para análise.

-showcerts: Adicionado ao comando acima, ele mostra não apenas o certificado do site, mas toda a cadeia de certificados (intermediários e raiz).

2. Anatomia e Campos do Certificado (X.509 v3)
Ao inspecionar um certificado real com openssl x509 -text, os seguintes campos são cruciais:

Serial Number: Identificador único dentro daquela Autoridade Certificadora (CA).

Issuer (Emissor): Quem assinou o certificado (ex: Let's Encrypt, GlobalSign).

Validity (Validade): O intervalo de tempo em que o certificado é considerado ativo.

Basic Constraints (CA: FALSE): Indica que aquele certificado é de "entidade final" (um site) e não pode ser usado para emitir outros certificados. Isso evita ataques de falsificação de cadeia.

Signature Algorithm: Mostra como a CA assinou o arquivo (geralmente um hash como SHA-256 criptografado com a chave privada da CA).

3. Conceitos de Confiança e Transparência
O vídeo aprofunda em como a segurança da web é mantida em larga escala:

Cadeia de Confiança (Chain of Trust)
Navegadores não conhecem todos os sites, mas confiam em CAs Raiz.

O Certificado do Servidor é assinado por uma CA Intermediária.

A CA Intermediária é assinada pela CA Raiz.

A CA Raiz está pré-instalada no seu computador/navegador.
Isso reduz o risco: se uma chave intermediária vazar, apenas uma parte dos certificados é afetada, protegendo a chave mestre (Raiz).

Certificate Transparency (CT)
Mecanismo onde todos os certificados emitidos devem ser registrados em logs públicos.

SCT (Signed Certificate Timestamp): Prova que o certificado está no log.

crt.sh: Ferramenta online mencionada para buscar todos os certificados já emitidos para um domínio, ajudando a detectar emissões fraudulentas.

4. Revogação: CRL vs. OCSP
Se um certificado for roubado antes de expirar, ele precisa ser cancelado.

CRL (Certificate Revocation List): Uma lista "negra" completa que o navegador baixa (legado, lento).

OCSP (Online Certificate Status Protocol): Uma consulta rápida em tempo real para saber se aquele certificado específico ainda é válido (mais moderno).

# PART 5 Certificados inválidos: #
1. Identificando Erros Comuns
O OpenSSL exibe os dados brutos, mas não avisa se algo está errado com letras vermelhas ou alertas. O engenheiro deve saber onde olhar:

A. Certificado Expirado
O navegador bloqueia o acesso, mas o OpenSSL apenas mostra as datas.

Comando alvo: openssl x509 -in cert.pem -noout -dates

O que verificar: As datas notBefore (início) e notAfter (fim). Se a data atual estiver fora desse intervalo, o certificado é inválido.

B. Nome de Host Incorreto (Wrong Hostname)
Ocorre quando o URL que você digita não consta no certificado.

O Problema do Wildcard: Um certificado para *.badssl.com protege site.badssl.com, mas não protege nivel3.site.badssl.com. Cada ponto extra no domínio exige um novo nível de validação.

Comando alvo: openssl x509 -in cert.pem -noout -text | grep -A1 "Subject Alternative Name"

C. Certificado Autoassinado (Self-Signed)
A criptografia existe, mas a identidade não foi validada por uma autoridade (CA) confiável.

Como identificar: Verifique se o emissor e o dono são a mesma pessoa.

Comando alvo: openssl x509 -in cert.pem -noout -subject -issuer

Resultado: Se subject == issuer, o certificado é autoassinado.

2. Técnicas Avançadas de Terminal (Linux)
Para agilizar o diagnóstico, você pode "encadear" comandos (Piping) para não precisar salvar arquivos temporários:

openssl s_client -connect self-signed.badssl.com:443 2>/dev/null | openssl x509 -noout -subject -issuer

s_client: Busca o certificado do site.

2>/dev/null: Esconde mensagens de erro irrelevantes do terminal.

| (Pipe): Envia o certificado direto para o próximo comando.

x509: Processa e exibe apenas o subject e o issuer.

3. Curiosidade: Certificados Gigantes
O vídeo explora um caso extremo: um certificado com 1.000 nomes alternativos (SANs).

Enquanto um certificado do Google tem cerca de 4.000 caracteres, este tem 40.000.

Isso serve para testar o limite de processamento dos clientes (navegadores e dispositivos IoT), que podem travar ou ficar lentos ao processar uma lista tão vasta.

# PART 6 Problemas de certificado: #
1. Ferramentas de Inspeção do Lado do Cliente
Para diagnosticar o que acontece "por baixo do capô" quando um cliente tenta se conectar, o vídeo introduz duas ferramentas trabalhando juntas:

ssl_dump: Funciona como o tcpdump ou Wireshark, mas é especializado em tráfego SSL/TLS. Ele captura os pacotes na placa de rede e decodifica as mensagens do handshake.

Comando comum: ssl_dump -nn -n -h -i eth0 (captura na interface eth0 com cabeçalhos detalhados).

openssl s_client: Já visto antes, aqui ele é usado para "forçar" comportamentos específicos e observar como o servidor reage.

2. Anatomia do Handshake (O Aperto de Mão)
O vídeo demonstra as diferenças vitais entre as versões do protocolo:

TLS 1.2 vs TLS 1.3
O TLS 1.3 é mais rápido e seguro, reduzindo o número de mensagens trocadas. No entanto, o TLS 1.2 ainda é muito usado para compatibilidade.

ClientHello: A primeira mensagem enviada pelo cliente. Nela, o cliente diz ao servidor: "Eu falo estas versões de TLS e suporto estas 30-40 Cipher Suites (algoritmos de criptografia)".

ServerHello: O servidor responde escolhendo a melhor opção comum. Se o cliente só oferecer cifras antigas e o servidor só aceitar novas, a conexão falha aqui.

3. Conceitos Cruciais de Segurança
SNI (Server Name Indication): Permite que um único endereço IP hospede vários certificados SSL (vários sites). O cliente envia o nome do site que deseja visitar logo no início da conversa para que o servidor saiba qual certificado mostrar.

ECDHE (Key Exchange): Método de troca de chaves onde cliente e servidor criam uma chave secreta compartilhada sem nunca enviá-la pela rede. Isso garante a Forward Secrecy (se a chave do servidor vazar no futuro, as conversas de hoje continuam seguras).

Master Session Key: O ssl_dump mostra que cada conexão gera uma chave mestre única. Mesmo que você se conecte ao mesmo site duas vezes seguidas, a chave de criptografia será diferente.

4. Troubleshooting Prático
A recomendação para resolver erros de SSL no cliente é abrir dois terminais:

No Terminal A, rode o ssl_dump para monitorar os pacotes.

No Terminal B, tente a conexão com openssl s_client -connect site:443.

Se a conexão cair, o ssl_dump mostrará exatamente em qual etapa (ex: após o ClientHello) o servidor enviou um "Alert" de fechamento.


# PRÁTICA:# 

Para entender como tudo isso se conecta na prática, imagine que você é o Administrador de Redes de uma empresa chamada "MinhaEmpresa". Você precisa colocar um site novo no ar (loja.minhaempresa.com.br) com segurança máxima.

Aqui está o fluxo real de trabalho, do zero até a monitoração:

Fase 1: O Nascimento da Identidade (Segurança do Lado do Servidor)
Antes de ter um certificado, você precisa de uma "identidade".

Geração da Chave Privada: Você escolhe entre RSA (compatibilidade) ou ECC (performance). Na prática, hoje usamos ECC por ser mais rápida.

Comando: openssl genpkey -algorithm EC...

Criação da CSR (O Pedido): Você gera a Solicitação de Assinatura de Certificado. Aqui você insere o Common Name (CN) como loja.minhaempresa.com.br.

Comando: openssl req -new -key chave.key -out pedido.csr

A Validação: Você envia o arquivo .csr para uma Autoridade Certificadora (CA) como a Digicert ou Let's Encrypt. Eles verificam se você é dono do domínio e te devolvem o Certificado Assinado (loja.crt).

Fase 2: A Implementação e Organização
Você recebeu o arquivo, mas o seu servidor Apache ou Nginx não inicia porque você misturou os arquivos antigos com os novos.

O Trabalho de Detetive (Matching): Você tem 5 chaves e 3 certificados na pasta. Como saber qual é o par?

Você usa o Modulus (para RSA) ou o Public Key (para ECC) e compara os hashes.

Comando: openssl pkey -in chave.key -pubout | openssl md5 comparado com o do certificado.

Instalação: Agora você aponta no seu servidor web:

SSLCertificateFile -> loja.crt

SSLCertificateKeyFile -> chave.key

Fase 3: Validação e Teste Real
O site está no ar, mas alguns usuários reclamam que aparece "Conexão Não Segura".

Simulando o Navegador: Você usa o terminal para ver o que o servidor está enviando de verdade, sem o "cache" do navegador atrapalhar.

Comando: openssl s_client -connect loja.minhaempresa.com.br:443

Checando a Cadeia (Chain of Trust): Você percebe que esqueceu de instalar o Certificado Intermediário. O s_client te mostra que o navegador não consegue chegar até a "Raiz" de confiança.

Correção: Você agrupa o seu certificado com o intermediário da CA.

Fase 4: Troubleshooting Avançado (Lado do Cliente)
Um cliente específico (um sistema antigo de um fornecedor) não consegue conectar no seu site novo.

Análise de Protocolo: Você usa o ssl_dump enquanto tenta conectar.

O Diagnóstico: No terminal, você vê que o cliente envia um ClientHello pedindo TLS 1.0 (antigo e inseguro), mas seu servidor só aceita TLS 1.2 ou 1.3.

A Decisão: Agora você tem uma prova técnica para dizer ao fornecedor: "Seu sistema está usando um protocolo de 1999, você precisa atualizar para o TLS 1.2".
