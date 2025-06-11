# APIs-REST

Você deverá criar um programa de console em C# que faça requisições periódicas a um serviço REST e apresente, a cada leitura, a temperatura no horário exato, indicando se a temperatura subiu, desceu ou permaneceu igual em relação à leitura anterior. A saída deve usar cores no console: vermelho para “subiu”, azul para “desceu” e cor padrão para “sem alteração”.



Ao iniciar, o programa deve solicitar ao usuário: A unidade de temperatura desejada (Celsius, Kelvin ou Fahrenheit).
O intervalo em segundos para efetuar cada nova leitura.

Leitura Periódica

Após obter a unidade e o intervalo, o programa deve entrar em um laço que se repete indefinidamente (ou até o usuário interromper), fazendo requisições HTTP GET ao endpoint:http://localhost:5000/temperatura/{unidade} onde “{unidade}” é a escolha do usuário (celsius, kelvin ou fahrenheit).Cada resposta retornará um JSON contendo o valor da temperatura na unidade solicitada.

Registro de Horário
Para cada requisição bem-sucedida, capture o horário local exato (hora, minuto e segundo) em que a resposta foi obtida.

Comparação com Leitura Anterior

O programa deve manter em memória apenas o valor da última temperatura recebida.
Na primeira leitura, não há comparação; basta exibir o resultado normalmente.

Da segunda leitura em diante, compare o valor atual com o valor da leitura imediatamente anterior: 
Se o valor atual for maior que o anterior, considere “temperatura subiu”.
Se o valor atual for menor que o anterior, considere “temperatura desceu”.
Se for igual, considere “sem alteração”.

Saída no Console

A cada leitura, imprima uma linha contendo:
O horário da leitura no formato HH:mm:ss.
O valor da temperatura (com duas casas decimais) seguido da unidade escolhida.

Uma indicação de variação (“SUBIU”, “DESCEU” ou “SEM ALTERAÇÃO”) colorida:
Vermelho para “SUBIU”.
Azul para “DESCEU”.
Cor neutra (padrão) para “SEM ALTERAÇÃO”.

Exemplo ilustrativo (somente texto):

[14:07:32] Temperatura: 27,43 °C → SUBIU 
[14:07:37] Temperatura: 27,10 °C → DESCEU 
[14:07:42] Temperatura: 27,10 °C → SEM ALTERAÇÃO 
[14:07:47] Temperatura: 28,02 °C → SUBIU

Tratamento de Erros

Se a requisição HTTP falhar ou o serviço não estiver disponível, o programa deve exibir uma mensagem de erro (“Erro ao obter leitura”) em cor amarela (ou cor padrão, caso o console não suporte cores) e continuar aguardando o próximo ciclo.
Se a resposta JSON estiver em formato inesperado, exibir “Resposta inválida” e seguir adiante.

Interrupção do Programa

Permita que o usuário interrompa a execução a qualquer momento (por exemplo, pressionando Ctrl+C).
Ao receber o comando de encerramento, exiba uma mensagem simples (por exemplo: “Encerrando monitoramento de temperatura.”) e finalize a execução.

Servidor REST
Abaixo um exemplo mínimo que expõe um endpoint GET para /temperatura/{unidade}, onde {unidade} pode ser "celsius", "kelvin" ou "fahrenheit". O servidor calcula, a cada requisição,
TC=25+5 sen(t 2/24) + ruído
sendo “t” o número de horas (incluindo fração) desde meia-noite. Em seguida converte para a unidade desejada e retorna um JSON simples.
Passo a passo
1.          Criar o projeto vazio No terminal:
mkdir ServidorTemp
cd ServidorTemp
dotnet new webapi -n ServidorTemp
cd ServidorTemp
2.  Limpar o conteúdo gerado Apague tudo que existe em Program.cs, WeatherForecast.cs e na pasta Controllers/. (O único arquivo que restará será Program.cs.)
3.          Copiar o código abaixo em Program.cs	
using System;

 var builder = WebApplication.CreateBuilder(args);
 var app = builder.Build();

 // unidade: "celsius", "kelvin" ou "fahrenheit"
 app.MapGet("/temperatura/{unidade}", (string unidade) =>
 {
 	// 1) obter hora atual em horas totais (ex.: 14.5 = 14h30m)
 	double t = DateTime.Now.TimeOfDay.TotalHours;

 	// 2) calcular temperatura em Celsius sem ruído
 	double tempCBase = 25.0 + 5.0 * Math.Sin((2.0 * Math.PI / 24.0) * t);

 	// 3) adicionar ruído uniforme [0,1)
 	double ruido = Random.Shared.NextDouble();
 	double tempC = tempCBase + ruido;

 	// 4) converter para a unidade pedida
 	double resultado;
 	string uni = unidade.ToLower();
 	if (uni == "kelvin")
 	{
     		resultado = tempC + 273.15;
 	}
 	else if (uni == "fahrenheit")
 	{
     		resultado = tempC * 9.0 / 5.0 + 32.0;
 	}
 	else if (uni == "celsius")
 	{
     		resultado = tempC;
 	}
 	else
 	{
     		return Results.BadRequest(new { erro = "Unidade inválida. Use celsius, kelvin ou fahrenheit." });
 	}

 	// 5) retornar JSON simples: { "unidade": "...", "valor": <número> }
 	return Results.Ok(new
 	{
     	unidade = uni,
     	valor = Math.Round(resultado, 2)
 	});
 });

 app.Run();
4.          Executar o servidor No terminal, dentro da pasta ServidorTemp:
dotnet run
Você verá algo como:
Now listening on: http://localhost:5000
Now listening on: https://localhost:5001
5.          Testar com curl (ou Postman)
–            Celsius:
curl http://localhost:5000/temperatura/celsius
Exemplo de resposta:
{
   "unidade": "celsius",
   "valor": 29.43
}
–            Kelvin:
  	
curl http://localhost:5000/temperatura/kelvin

Retorno aproximado:
{
   "unidade": "kelvin",
   "valor": 302.58
}
–            Fahrenheit:
  	
curl http://localhost:5000/temperatura/fahrenheit

Retorno aproximado:
{
   "unidade": "fahrenheit",
   "valor": 87.17
}
               Se a {unidade} não for uma das três opções, retorna HTTP 400 com:
  
{
   "erro": "Unidade inválida. Use celsius, kelvin ou fahrenheit."
}
Explicação mínima do que acontece
1.    DateTime.Now.TimeOfDay.TotalHours: recupera o horário atual como um número de horas (por exemplo, 14.25 = 14h15m).
2.       25 + 5 * sin((2π/24) * t): gera uma onda senoidal diária (um “bump” ao longo de 24 h) com média 25 °C e amplitude 5 °C.
3.          Random.Shared.NextDouble(): adiciona ruído uniforme entre 0 e 1.
4.          Fazemos conversão de tempC para “kelvin” ou “fahrenheit” conforme a rota.
5.          Retornamos um objeto anônimo em JSON contendo { unidade, valor }.
Estrutura final do diretório
ServidorTemp/
 ├── Program.cs
 └── ServidorTemp.csproj


