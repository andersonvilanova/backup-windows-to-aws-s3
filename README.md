# Backup de Pastas e Arquivos Windows para o AWS S3
Aqui será mostrado passo a passo como realizar a configuração de um usuário com permissões de gravar apenas em um bucket específico, e como criar a task no Windows que fará a cópia de uma pasta diariamente para o AWS S3, por meio do AWS CLI.

# Criar o bucket
1. Dentro da console da AWS, vá em S3 e escolha a opção "Create Bucket"
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/052ce205-38db-4c3c-bed1-95e1566c0f19)

2. Escolha o nome do bucket e a região que ficará localizado. O restante das opções mantive padrão.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/2e40daa0-7075-49f6-a58b-af60bb1a298d)
   
3. Anote o nome do bucket criado, pois será necessário ao criar o usuário que terá permissões de manipular os dados dentro deste bucket.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/6d986adc-54a1-49cd-8277-c8ef47a5d3bb)

# Criar a Policy
4. Agora acesse o AWS IAM, para criar uma nova policy.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/bc3afccf-c9d4-442f-b18a-0952d7183ea6)

5. Clique na opção JSON, e cole o código abaixo, substituindo "BUCKET-NAME", com o nome do bucket que você criou. Em seguida clique em Next.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                        "s3:GetBucketLocation",
                        "s3:ListAllMyBuckets"
                      ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::BUCKET-NAME",
                "arn:aws:s3:::BUCKET-NAME/*"
            ]
        }
    ]
}
```
Observe na imagem abaixo como ficou com o bucket que foi criado para esse exemplo.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/039e71c4-b38f-4244-b0cd-bb008fc90bec)

6. Na tela Policy details, digite um nome para a policy criada, e clique em Create Policy.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/0dd85041-7416-41b2-8fc1-ae4041196e72)

# Criar o usuário

7. Agora vá em AWS IAM, e selecione a opção "Create User".
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/956cdc2c-a99d-46c9-856a-32123ff08868)

8. Digite o nome do usuário e clique em "Next".
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/5875671d-d82d-4461-adb5-02aeff062857)

9. Na tela Set permissions, selecione a opção Attach policies directly, e pesquise pela Policy criada. Uma vez selecionada, clique em Next. Na tela de revisão, clique em Create User.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/a56e0bb3-c8ec-4689-bd0f-dcb8efa0606f)

# Gerar uma Access Key e Secret Access Key

10. Agora clique no usuário e na aba Security credentials, clique em Create access key. Você deverá informar a função da chave. Selecione Other. Em seguida clique em Next, e na próxima tela, clique em Create access key.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/64159377-5c3e-4735-a5f1-005753240af4)
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/4df1ec41-3c2f-4c69-b9db-f8f28ba007a6)

11. Anote os dados da Access key e Secret access key em local seguro. Outra opção é realizar o download do csv, que conterá esses dados.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/e4c46e66-4cd6-4878-b70c-ec31f7eeb06f)

# Configurar o servidor Windows para realizar o backup

12. Aqui não será demonstrado como instalar o AWS CLI, pois o procedimento é simples e consta no próprio site da AWS. Mas se ainda não baixou pode acessar o link https://awscli.amazonaws.com/AWSCLIV2.msi para fazer o download, ou acessar o link https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html para obter instruções mais detalhadas do instalador. Para instalar basta seguir o famoso NNF (Next, Next, Finish =D).

13. Agora abra o Prompt de comando do Windows, e digite aws configure. Você será solicitado a ingressar com alguns dados. Preencha os campos AWS Access Key ID e AWS Secret Access Key com os dados que você copiou quando gerou esses dados do usuário. O campo Default region name, deve ser o código da região AWS que você configurou o seu bucket S3. Acesse o link https://docs.aws.amazon.com/pt_br/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html se desejar consultar os códigos das regiões AWS. O campo Default output format não precisa ser preenchido. Observe a tela a seguir.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/fbf8ebbe-2fc0-498e-8e37-c64e15502e04)
    
14. Crie uma pasta chamada Scripts (ou outra que preferir) na raiz do disco C do servidor. Então crie um arquivo .bat com o seguinte comando (lembrando que C:\Backup você deve substituir pelo caminho que contém os arquivos que você deseja realizar o backup, e backup-windows-server/backup pelo nome do seu bucket, sendo que o /backup é porque eu quero colocar o conteúdo do meu backup dentro de uma pasta chamada backup no bucket. Eu não quero que o backup seja copiado direto na raiz do bucket).
```
aws s3 cp --recursive "C:\Backup" s3://backup-windows-server/backup
```
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/3829e8f0-6104-470f-a28b-3875532dca2b)

15. Agora crie uma task no Windows Server, utilizando o Task Screduler, especificando o caminho da .bat e o agendamento de sua preferência. Observe nas imagens abaixo, o passo a passo na criação da task.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/0f6d5cd8-d287-4619-adf6-bea626a8d591)
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/b43a3f89-6cd9-45b9-9a0b-6e70d332a176)
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/cb4d0601-0632-4331-a90f-94b9f25af129)
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/147b22ab-82d2-43d3-8c2d-0e29521bb8e4)
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/00d2e016-aa4c-49e4-a07c-f08373e5fae4)
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/dc333338-5176-4606-8fbf-34e6b6b8d4f3)
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/a983e75f-1d7b-4b70-bf6b-eedb0c29eeb1)

16. Aguarde a execução da Task e confirme que os dados foram copiados para o bucket S3.
![image](https://github.com/andersonvilanova/backup-windows-to-aws-s3/assets/37702307/21b34f9a-66e7-415a-9aaf-ae240c335671)
