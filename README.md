# JSON4Delphi

Biblioteca leve para manipulação de **JSON** em **Delphi** (inclusive
versões antigas) e **Lazarus/FreePascal (FPC 3.0+)**.

-   Implementação em **Object Pascal nativo**.\
-   Não depende de RTTI nem de bibliotecas externas.\
-   Usa apenas classes básicas (`TList`, `TStrings`, `TStringList`).

------------------------------------------------------------------------

## 📦 Instalação

Basta incluir a unidade `jsons.pas` no seu projeto:

``` pascal
uses jsons;
```

Compatível com **Delphi 6+** e **Lazarus/FPC**.

------------------------------------------------------------------------

## 🏗 Estrutura de Classes

### `TJsonObject` / `TJson`

Representa um objeto JSON (chave/valor).

-   `Put(Key, Value)` → adiciona/atualiza campo.\
-   `Values['Key']` → acessa valores.\
-   Propriedades de conversão: `AsString`, `AsInteger`, `AsBoolean`,
    `AsObject`, `AsArray`.

### `TJsonArray`

Representa um array JSON.

-   `Put(Value)` → adiciona item.\
-   `Items[I]` → acessa valores por índice.\
-   `Count` → quantidade de itens.

------------------------------------------------------------------------

## ✍️ Criando JSONs

``` pascal
var
  Json: TJson;
  Str: String;
begin
  Json := TJson.Create();

  Json.Put('field1', null);
  Json.Put('field2', True);
  Json.Put('field3', 3.14);
  Json.Put('field4', 'hello world');

  Json['field5'].AsBoolean := False;
  Json['field6'].AsString := 'hello world';

  // Subobjeto
  with Json['field7'].AsObject do
  begin
    Put('subfield1', 2.7182818284);
    Put('subfield2', 'json4delphi');
  end;

  // Array
  with Json['field8'].AsArray do
  begin
    Put(6.6260755e-34);
    Put('The magic words are squeamish ossifrage');
  end;

  // Converter para string JSON
  Str := Json.Stringify;

  Json.Free;
end;
```

📌 Resultado:

``` json
{
  "field1": null,
  "field2": true,
  "field3": 3.14,
  "field4": "hello world",
  "field5": false,
  "field6": "hello world",
  "field7": {
    "subfield1": 2.7182818284,
    "subfield2": "json4delphi"
  },
  "field8": [
    6.6260755e-34,
    "The magic words are squeamish ossifrage"
  ]
}
```

------------------------------------------------------------------------

## 📥 Parsing de JSON

``` pascal
var
  Json: TJsonObject;
begin
  Json := TJsonObject.Create;
  try
    Json.Parse('{"a":1,"b":"teste"}');

    ShowMessage(Json.Values['a'].AsString); // 1
    ShowMessage(Json.Values['b'].AsString); // teste
  finally
    Json.Free;
  end;
end;
```

------------------------------------------------------------------------

## 📚 Arrays e Objetos Aninhados

``` pascal
var
  lJson, lCredenciais: TJsonObject;
  lAdicionais: TJsonArray;
begin
  lJson := TJsonObject.Create(nil);
  lCredenciais := TJsonObject.Create(nil);
  lAdicionais := TJsonArray.Create(nil);
  try
    lCredenciais.Put('Email', 'usuario@teste.com');
    lCredenciais.Put('Senha', '123456');
    lJson.Put('Credenciais', lCredenciais);

    lJson.Put('Documento', '12345678900');

    lAdicionais.Put(2);
    lAdicionais.Put(3);
    lAdicionais.Put(4);
    lJson.Put('Adicionais', lAdicionais);

    ShowMessage(lJson.Stringify);
  finally
    lJson.Free;
    lCredenciais.Free;
    lAdicionais.Free;
  end;
end;
```

📌 Saída:

``` json
{
  "Credenciais": {
    "Email": "usuario@teste.com",
    "Senha": "123456"
  },
  "Documento": "12345678900",
  "Adicionais": [2,3,4]
}
```

------------------------------------------------------------------------

## 🔄 Iterando Arrays

``` pascal
var
  Arr: TJsonArray;
  I: Integer;
begin
  Arr := Json.Values['PendenciasFinanceiras']
            .AsObject.Values['PendenciasFinanceirasDetalhe']
            .AsArray;

  for I := 0 to Arr.Count - 1 do
    ShowMessage(Arr.Items[I].AsObject.Values['Valor'].AsString);
end;
```

------------------------------------------------------------------------

## 🚀 Exemplo Avançado (Parse de API)

``` pascal
procedure TfrmPrincipal.ParseSerasaJson(const AJsonStr: String);
var
  Json, Obj, ObjDet: TJsonObject;
  Arr: TJsonArray;
  I: Integer;
begin
  Json := TJsonObject.Create;
  try
    Json.Parse(AJsonStr);

    if Assigned(Json.Values['RiskScore']) then
    begin
      Obj := Json.Values['RiskScore'].AsObject.Values['PessoaFisica'].AsObject;
      edtScore.Text := Obj.Values['Pontuacao'].AsString;
      edtMsgScore.Text := Obj.Values['Descricao'].AsString;
    end;

    if Assigned(Json.Values['PendenciasFinanceiras']) then
    begin
      Arr := Json.Values['PendenciasFinanceiras'].AsObject
                .Values['PendenciasFinanceirasDetalhe'].AsArray;

      for I := 0 to Arr.Count - 1 do
      begin
        ObjDet := Arr.Items[I].AsObject;
        lbxPendFinanceiras.Items.Add('Data: ' + ObjDet.Values['DataOcorrencia'].AsString);
        lbxPendFinanceiras.Items.Add('Valor: ' + ObjDet.Values['Valor'].AsString);
      end;
    end;
  finally
    Json.Free;
  end;
end;
```

------------------------------------------------------------------------

## ✅ Boas Práticas

-   Sempre usar `try...finally` para liberar memória.\
-   Usar `Assigned` antes de acessar objetos/arrays opcionais.\
-   Utilizar `AsString`, `AsInteger`, `AsBoolean` conforme o tipo
    esperado.\
-   Para performance em grandes JSONs, evite `Stringify` a cada
    modificação (serialize no final).

------------------------------------------------------------------------

## 📌 Cheat Sheet (Referência Rápida)

  ----------------------------------------------------------------------------------------------------
  Operação     Exemplo                                                            Descrição
  ------------ ------------------------------------------------------------------ --------------------
  Criar objeto `Json := TJsonObject.Create;`                                      Instancia objeto
                                                                                  JSON.

  Adicionar    `Json.Put('Nome','Melissa');`                                      Insere valor string.
  campo                                                                           

  Atribuição   `Json['Ativo'].AsBoolean := True;`                                 Define valor
  direta                                                                          booleano.

  Subobjeto    `Json['Endereco'].AsObject.Put('Cidade','SP');`                    Cria/edita objeto
                                                                                  dentro.

  Array        `Json['Tags'].AsArray.Put('Dev');`                                 Insere item em
                                                                                  array.

  Iterar array `for I:=0 to Arr.Count-1 do ShowMessage(Arr.Items[I].AsString);`   Loop em elementos.

  Acessar      `Json.Values['Nome'].AsString`                                     Obtém valor por
  campo                                                                           chave.

  Parse JSON   `Json.Parse('{"a":1}');`                                           Converte string →
                                                                                  objeto.

  Stringify    `Str := Json.Stringify;`                                           Converte objeto →
                                                                                  string JSON.

  Limpar       `Json.Clear;`                                                      Remove todos os
                                                                                  valores.

  Liberar      `Json.Free;`                                                       Libera memória.
  ----------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

