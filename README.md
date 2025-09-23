# Entrega do Projeto — Modelo de Previsão em C# (ML.NET)

> Este repositório serve como exemplo completo de entrega em **C# com ML.NET**: inclui o passo-a-passo para treinar um modelo, expô-lo via **Minimal API**, e um arquivo `endpoints.json` com a descrição dos pontos de extremidade.

---

## Resumo

Neste README você encontrará instruções claras e reproduzíveis para:

1. Criar e treinar um modelo com **ML.NET**.
2. Salvar o artefato do modelo (`MLModel.zip`).
3. Expor endpoints HTTP (`/health` e `/predict`) usando **Minimal API** em .NET 6/7.
4. Gerar e salvar um arquivo `endpoints.json` descrevendo os pontos de extremidade.
5. Publicar o projeto no GitHub.

---

## Estrutura sugerida do repositório

```
modelo-previsao-entrega-csharp/
├─ README.md
├─ endpoints.json
├─ MinimalApiML/
│  ├─ Program.cs
│  ├─ ModelInput.cs
│  ├─ ModelOutput.cs
│  ├─ MLModel.zip        # modelo treinado salvo
│  └─ MinimalApiML.csproj
└─ TrainModel/
   ├─ TrainModel.cs
   └─ TrainModel.csproj
```

---

## 1) Criar o repositório no GitHub

1. No GitHub, clique em **New repository** → dê um nome (ex.: `modelo-previsao-entrega-csharp`).
2. Clone localmente ou inicialize git no diretório do projeto:

```bash
git init
git add .
git commit -m "Initial commit: modelo ML.NET + endpoints"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/NOME_REPOSITORIO.git
git push -u origin main
```

---

## 2) Treinar e salvar o modelo

Criar um projeto de console para treinar e salvar o modelo:

```bash
dotnet new console -o TrainModel
cd TrainModel
dotnet add package Microsoft.ML
```

👉 `TrainModel.cs`

```csharp
using Microsoft.ML;
using Microsoft.ML.Data;

public class ModelInput
{
    public float Feature1 { get; set; }
    public float Feature2 { get; set; }
    public float Feature3 { get; set; }
    public float Label { get; set; }
}

public class ModelOutput
{
    [ColumnName("Score")]
    public float Prediction { get; set; }
}

class Program
{
    static void Main()
    {
        var mlContext = new MLContext();

        var data = mlContext.Data.LoadFromEnumerable(new[]
        {
            new ModelInput { Feature1 = 1, Feature2 = 2, Feature3 = 3, Label = 10 },
            new ModelInput { Feature1 = 2, Feature2 = 3, Feature3 = 4, Label = 15 },
            new ModelInput { Feature1 = 3, Feature2 = 4, Feature3 = 5, Label = 20 }
        });

        var pipeline = mlContext.Transforms.Concatenate("Features", "Feature1", "Feature2", "Feature3")
            .Append(mlContext.Regression.Trainers.Sdca());

        var model = pipeline.Fit(data);

        mlContext.Model.Save(model, data.Schema, "../MinimalApiML/MLModel.zip");

        Console.WriteLine("Modelo treinado e salvo em MLModel.zip");
    }
}
```

Rodar o treino:

```bash
dotnet run --project TrainModel
```

Isso gerará o arquivo `MLModel.zip` na pasta `MinimalApiML/`.

---

## 3) Minimal API com ML.NET

Criar o projeto da API:

```bash
dotnet new web -o MinimalApiML
cd MinimalApiML
dotnet add package Microsoft.ML
```

👉 `ModelInput.cs`

```csharp
public class ModelInput
{
    public float Feature1 { get; set; }
    public float Feature2 { get; set; }
    public float Feature3 { get; set; }
}
```

👉 `ModelOutput.cs`

```csharp
public class ModelOutput
{
    public float Prediction { get; set; }
}
```

👉 `Program.cs`

```csharp
using Microsoft.ML;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var mlContext = new MLContext();
DataViewSchema modelSchema;
var mlModel = mlContext.Model.Load("MLModel.zip", out modelSchema);
var predEngine = mlContext.Model.CreatePredictionEngine<ModelInput, ModelOutput>(mlModel);

app.MapGet("/health", () => new { status = "ok" });

app.MapPost("/predict", (ModelInput input) =>
{
    var result = predEngine.Predict(input);
    return Results.Ok(new { prediction = result.Prediction });
});

app.Run();
```

---

## 4) Endpoints

👉 `endpoints.json`

```json
{
  "project": "modelo-previsao-entrega-csharp",
  "version": "1.0",
  "endpoints": [
    {
      "name": "health",
      "path": "/health",
      "method": "GET",
      "description": "Verifica se a API está no ar",
      "response_example": { "status": "ok" }
    },
    {
      "name": "predict",
      "path": "/predict",
      "method": "POST",
      "description": "Recebe JSON com features e retorna previsão",
      "request_example": { "feature1": 1.0, "feature2": 2.0, "feature3": 3.0 },
      "response_example": { "prediction": 12.34 }
    }
  ]
}
```

---

## 5) Rodar a API

No terminal:

```bash
dotnet run --project MinimalApiML
```

Testes:

* [http://localhost:5000/health](http://localhost:5000/health)
* POST em `/predict` com JSON:

```json
{
  "feature1": 1.0,
  "feature2": 2.0,
  "feature3": 3.0
}
```

---

## 6) Checklist de entrega

* [ ] Repositório criado no GitHub (link público)
* [ ] `README.md` com o passo-a-passo
* [ ] `endpoints.json` salvo
* [ ] `MLModel.zip` gerado e incluído
* [ ] API executando localmente


---

