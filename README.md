# -APEX-download-de-arquivo-bot-o-ou-link
download de arquivo botão ou link
```
create or replace PROCEDURE get_id (p_get_id  IN VARCHAR2) IS
  l_blob_content  CELL_BOLETO.CONTENT%TYPE;
  l_mime_type     CELL_BOLETO.MIMETYPE%TYPE;
  l_file_name     CELL_BOLETO.FILENAME%TYPE;
BEGIN
  SELECT CONTENT,
         MIMETYPE,
         FILENAME
  INTO   l_blob_content,
         l_mime_type,
         l_file_name
  FROM   CELL_BOLETO
  WHERE  ID_BOLETO = p_get_id;
  sys.HTP.init;
  sys.OWA_UTIL.mime_header(l_mime_type, FALSE);
  sys.HTP.p('Content-Length: ' || DBMS_LOB.getlength(l_blob_content));
  sys.HTP.p('Content-Disposition: filename="' || l_file_name || '"');
  sys.OWA_UTIL.http_header_close;
  sys.WPG_DOCLOAD.download_file(l_blob_content);
  apex_application.stop_apex_engine;
EXCEPTION
  WHEN apex_application.e_stop_apex_engine THEN
    NULL;
  WHEN OTHERS THEN
    HTP.p('ERRO NO ARQUIVO');
END;
```
# Configuração do APEX para Download de Arquivos

## Item de Aplicativo

1. Crie um novo item de aplicativo.

   - **Caminho:** Componentes compartilhados > Itens de aplicativo
   - **Ação:** Clique no botão "Criar".
   - **Detalhes do Item:**
      - **Nome:** FILE_ID
      - **Escopo:** Aplicação
      - **Proteção do estado da sessão:** Soma de verificação necessária - nível do usuário
   - Clique no botão "Criar item de aplicativo".

## Processo de Inscrição

2. Crie um novo processo de inscrição.

   - **Caminho:** Componentes Compartilhados > Processos de Aplicativo
   - **Ação:** Clique no botão "Criar".
   - **Detalhes do Processo:**
      - **Nome:** GET_FILE
      - **Sequência:** {aceitar o padrão}
      - **Ponto de processo:** Retorno de chamada do Ajax: execute este processo de aplicativo quando solicitado por um processo de página.
   - Clique no botão "Avançar".
   - Insira o PL/SQL necessário para realizar o download.
```
BEGIN 
  get_file(:FILE_ID); 
FIM;
```
   - Clique no botão "Avançar".
   - Selecione o tipo de condição "O usuário é autenticado (não público)".
   - Clique no botão "Criar Processo".
   - Se precisar adicionar autorização, clique no novo processo de inscrição, selecione o esquema de autorização e clique no botão "Aplicar alterações".

---

##Botão
 - No painel de propriedades, na seção “Comportamento”, selecione a “Ação” de “Redirecionar para URL”.
```
f?p=&APP_ID.:1:&APP_SESSION.:APPLICATION_PROCESS=GET_FILE:::FILE_ID:MEU_ID
```
##Link
```
<a href="f?p=&APP_ID.:1:&APP_SESSION.:APPLICATION_PROCESS=GET_FILE:::FILE_ID:MEU_ID">Baixar</a>
```
