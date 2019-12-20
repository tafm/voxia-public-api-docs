
# Transcrevendo a sua primeira mídia
---

### Antes de começar
1. Obtenha a ApiKey que irá permitir acesso da sua aplicação [Getting Started](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/c/Guides/Getting%20Started).
2. Obtenha  o token de autenticação que identificará e autenticará seu usuário nas chamadas à API na página de preferências de usuário no Voxia

### Faça requisição para criar uma nova Mídia local
1. Envie uma requisição POST para a api [/files/media](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/post).
Os parâmetros que devem ser enviados são:
    	{ 
		"name": "nome da sua transcricão2.flac", //Nome da sua mídia/arquivo
    	"source":{
    		"origin": "local", //A origem da mídia será do seu dispositivo local
    		"params": {
    			"fileType": "audio/flac" //Especifique o Mime Type do arquivo à ser enviado
    		}
    	},
    	"generateTranscription": true, // Escolha se você quer ou não que o seu arquivo seja transcrito automaticamente
    	"tags": [  //Pelo menos duas TAGs devem ser especificadas e a primeira deve possuir o valor "MPPE"
			"MPPE",  // TAG OBRIGATÓRIA, não alterar.
			"minha primeira tag" // É necessário especificar a segunda TAG e ela será indexada junto com a transcrição e o "name"
		],
    	"parent": null //ID da pasta em que a transcrição será enviada (null é a raíz do diretório)
		}
Para realizar a sua requisição com o cURL, copie, cole e altere as propriedades no seu terminal: (Lembre-se de colocar  [SUA-API-KEY] e [SEU-TOKEN])
```bash
curl -X POST -H 'Content-Type: application/json' -H "Authorization: [SEU-TOKEN]" -d '{"name":"nome da sua transcricão2.flac","source":{"origin":"local","params":{"fileType":"audio/flac"}},"generateTranscription":false,"tags":["MPPE","minha primeira tag"],"parent":null}' https://voxia-public-api-proxy-jwt-agskifmnha-uc.a.run.app/v1/files/media?key=[SUA-API-KEY]
```
2. Após executar o comando você deverá receber como resposta: 
    	{
    		"media": {
    			"id": "pXGvG3g43qx3lXeRzY5X232",
    			"name": "nome da sua transcricão.flac",
    			"createdAt": "2019-12-19T18:36:30.145Z",
    			"createdBy": "meuemail@email.com.br",
    			"parent": null,
    			"duration": null,
    			"finishedAt": null,
    			"metadata": {
    				  "NPU": "minha primeira tag",
    				  "tags": [
    					"MPPE",
    					"minha primeira tag"
    			],
    			  "process_type": "MPPE"
    		},
    		"status": "initing",
    		"thumbnails": [],
    		"generateTranscription": false,
    		"source": {
    			  "params": {
    				"fileType": "audio/flac"
    			},
    			"origin": "local"
    			}
    		},
    		"postMediaInfo": {
    		"uploadUri": "https://storage.googleapis.com/voxia-uploaded-direct/pXGvG3g43qx3lXeRzY5X?GoogleAccessId=mppe-3803f%40appspot.gserviceaccount.com&Expires=1576786590&Signature=uql9bWTa5eVE5CgejVONTI%2BHRLiIpO01%2B%2Fb78fxK%2FGwlhtMYi%2BDW9m%2F2wxNNj36ILvSyfSh6aDooRUDjiqikkL1ZtYQu%2"
    		 }
    	}
Salve o retorno da requisição. O parâmetro  `media.id` é o identificador único da sua mídia e o parâmetro `postMediaInfo.uploadUri` contem o link que será utilizado para o upload/envio da sua mídia.

### Conseguindo o ID de sessão de upload

No passo anterior você recebeu `postMediaInfo.uploadUri` com uma URL temporária para o upload do seu arquivo, porém é necessário ainda o ID do upload, um parâmetro de query, com ele será possível enviar parte de um arquivo em momentos (e até dispositivos) diferentes ou implementar funcionalidade de pause no upload.

1. Faça uma requisição POST para a url recebida em `postMediaInfo.uploadUri` passando os cabeçalhos "Content-Type" com o formato do arquivo e "x-goog-resumable" sempre com o valor "start"

```bash
curl -v -X POST -H 'Content-Type: audio/flac' -H 'x-goog-resumable: start' "[postMediaInfo.uploadUri]"
```
Caso o código de status retornado seja 201 (created), nos cabeçalhos retornados terá o `Location` com a URL de upload completa de upload, ou seja, o `postMediaInfo.uploadUri` com o parâmetro `upload_id`

### Faça o upload do arquivo da Mídia local

De posse da `Location` para upload o próximo passo é enviar o arquivo
1. Faça uma requisição PUT para a url recebida em `Location` enviando a sua mídia.
```bash
cat "[caminho-para-minha-mídia.mp4]" | curl -X PUT -H 'Content-Type: [formato do arquivo]' --data-binary @- "[Location]"
```

### Cheque se a mídia já foi transcrita

As transcrições automáticas levam em média metade do tempo de duração da mídia enviada. Esse tempo pode variar devido à fila de requisições. Depois de realizar o upload da mídia aguarde alguns instantes e comece à verificar se a transcrição da sua mídia foi finalizada. Para realizar essa verificação:

1. Envie uma requisição GET para a api [/files/media/{mediaId}](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/%7BmediaId%7D/get) em que `{mediaId}` é o ID da sua mídia retorna pela requisição POST inicial na API [/files/media](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/post).
`curl -X POST -H 'Content-Type: application/json' -d '{"source":{"origin"},"password":"something"}' https://voxia-public-api-esp-agskifmnha-uc.a.run.app/v1/files/media?key=[SUA-API-KEY]`
2. Caso o parâmetro `finishedAt` tenha o valor diferente de `null`, pode considerar que a transcrição da mídia foi finalizada.


### Exporte o resultado da transcrição da sua Mídia

Quando finalizada a transcrição, você pode realizar a exportação do que foi transcrito para diversos formatos: [DOCX](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/%7BmediaId%7D/export/docx/post), [PDF](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/%7BmediaId%7D/export/pdf/post), [JSON](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/%7BmediaId%7D/export/json/post), [subtitle](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/%7BmediaId%7D/export/subtitle/post), entre outros. Para exportar no formato JSON:
1. Envie uma requisição POST para a api [/files/media/{mediaId}/export/json](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/%7BmediaId%7D/export/json/post) em que `{mediaId}` é o ID da sua mídia retorna pela requisição POST inicial na API [/files/media](https://endpointsportal.audfacil.cloud.goog/docs/voxia-public-api-esp-agskifmnha-uc.a.run.app/1/routes/files/media/post).
`curl -X POST -H 'Content-Type: application/json' -d '{"source":{"origin"},"password":"something"}' https://voxia-public-api-esp-agskifmnha-uc.a.run.app/v1/files/media/{mediaId}/export/json?key=[SUA-API-KEY]`
2. Salve o JSON retornado com o array de parágrafos no seguinte formato:

```json
{
  "name": "minha transcrição",
  "content": [
    {
      "reviewed": false,
      "speaker": "",
      "content": [
        {
          "st": 0.2,
          "et": 2.4,
          "text": "Parágrafo"
        },
        {
          "st": 2.4,
          "et": 2.8,
          "text": " 1"
        }
      ]
    },
    {
      "reviewed": false,
      "speaker": "",
      "content": [
        {
          "st": 3.0,
          "et": 3.2,
          "text": "Parágrafo"
        },
        {
          "st": 3.7,
          "et": 4.0,
          "text": " 2"
        }
      ]
    }
  ]
}
```
