# Lambda AWS:.

## Serverless

````
Sem servidor: Para executar uma atividade rapidamente
Cobrado pelo numero de execuções da tarefa x tempo rodando
Uma função pode chamar outras
$ aws lambda invoke --function-name <nome-da-funcao> <arquivo-com-saida-da-funcao.txt> <- Para execuar pelo AWS CLI
As lambdas utilizão gatilhos para executar:
Amazon Event Bridge criar um gatilho que executa a lambda 1x por dia
S3 - Para manipular arquivos na AWS
````

### Gatilho: Event Bridge

Todo dia as 18h                                                                 <br>
![img.png](imgs/img.png)                                                             <br>
Ao acontecer chama a lambda                                                     <br>
![img_1.png](imgs/img_1.png)                                                         <br>

### Gatilho: Ao adicionar no S3

![img_2.png](imgs/img_2.png)                                                         <br>
![img_3.png](imgs/img_3.png)                                                         <br>
Agora quando recebe uma imagem ele executa a lambda, verificar no Cloudwatch.   <br>
Node: Executa o que chegou e imprime o log.                                     <br>
![img_4.png](imgs/img_4.png)                                                         <br>
Ajustando o consumo de memória, se executar mais tempo ele encerra.             <br>
![img_5.png](imgs/img_5.png)                                                         <br>

````
Consigo editar pelo site da aws ou gerar um zip e fazer upload pelo CLI
$ aws lambda update-function-code --function-name primeiraFuncao --zip-file fileb://lambda.zip
````

### GitHub Actions

![img_6.png](imgs/img_6.png)                                                         <br>
https://github.com/marketplace/actions/aws-lambda-deploy                        <br>
Mais utilizada:                                                                 <br>
Fazer o commit da lambda                                                        <br>
![img_7.png](imgs/img_7.png)                                                         <br>
Escondendo as chaves AWS - Settings - Secrets and variables - Actions           <br>
![img_8.png](imgs/img_8.png)                                                         <br>
Volte para actions e vamos criar a action - set up a workflow yourself          <br>
Agora quando faz o push ele já atualiza a lambda                                <br>
Quando utilizamos dependências externas, precisamos enviá-las junto com nossa aplicação, pois a AWS Lambda não as
instala automaticamente em suas runtimes. <br>

````
name: deploy to lambda
on: [push]
jobs:
  deploy_zip:
    name: deploy lambda function
    runs-on: ubuntu-latest
    steps:
      - name: checkout source code                      #Pegar o codigo
        uses: actions/checkout@v1
      - name: Generate ZIP                              #Gerar o ZIP
        run: |
          zip deployment.zip *.mjs
      - name: default deploy                            #Fazer o deploy da Master
        uses: appleboy/lambda-action@master
        with:
          aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws_region: ${{ secrets.AWS_REGION }}
          function_name: primeiraFuncao
          zip_file: deployment.zip
````

### Variaveis de Ambiente

![img_9.png](imgs/img_9.png)                                                         <br>
![img_10.png](imgs/img_10.png)                                                       <br>

### URL da função

![img_11.png](imgs/img_11.png)                                                       <br>
![img_12.png](imgs/img_12.png)                                                       <br>
Configurando a resposta no formato HTML                                         <br>
![img_13.png](imgs/img_13.png)                                                       <br>

### Runtime Personalizada - Atraves de uma imagem de container (Inicialização mais lenta para aquecer)

ECR - Envia a imagem para o repositório da AWS                                  <br>
![img_15.png](imgs/img_15.png)                                                       <br>
![img_16.png](imgs/img_16.png)                                                       <br>
![img_14.png](imgs/img_14.png)                                                       <br>
![img_17.png](imgs/img_17.png)                                                       <br>
![img_18.png](imgs/img_18.png)                                                       <br>
No debploy o Github Actions é diferente, precisa gerar uma nova imagem          <br>
No caso de lambdas Java podemos executar ela para deixar sempre ativa (hot), executando a proxima chamada mais
rápido<br>
Imagem do PHP:.                                                                 <br>
https://gist.github.com/CViniciusSDias/21b2073f73c9c928207d163c792e063c         <br>
![img_19.png](imgs/img_19.png)                                                       <br>

### Testando a lambda local

https://docs.aws.amazon.com/pt_br/serverless-application-model/?id=docs_gateway <br>
https://docs.aws.amazon.com/pt_br/serverless-application-model/latest/developerguide/what-is-sam.html <br>
https://www.serverless.com/framework/docs/getting-started                       <br>

### Simular um codigo que limita upload na lambda

````
import { log } from './log.mjs';
import { S3 } from '@aws-sdk/client-s3';                    <- Import direto da AWS

const s3Client = new S3({ region: 'us-east-1' });           <- Seta a região
export const handler = async(event) => {

    const record = event.Records[0];
    const Bucket = record.s3.bucket.name;
    const Key = record.s3.object.key;                       <- Busca o objeto no s3
    const getObjectResult = await s3Client.getObject({      <- Assincrono
        Bucket,
        Key,
    });
    const mega_byte = 1024 * 1024;                          <- Converte para Megabytes

    if (getObjectResult.ContentLength > 1 * mega_byte) {
        log('Objeto muito grande');
        return 'Objeto muito grande';
    }

    log('Objeto de tamanho OK');
    return 'Objeto de tamanho OK';
};
````

https://github.com/alura-cursos/aws-lambda-alura                                <br>
























