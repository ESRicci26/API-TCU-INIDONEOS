Consulta TCU - Certidões Inidôneos
Este projeto consiste em uma página web que permite consultar informações de certidões do Tribunal de Contas da União (TCU) de inidôneos a partir do CNPJ de uma empresa. Ele oferece a opção de baixar a certidão em formato PDF ou exibir os dados diretamente no navegador.
Exemplo CNPJ:
04268324000166
30139983000102

Funcionalidades
Consulta por CNPJ: Permite inserir um CNPJ para buscar informações de certidões.

Opção de Download em PDF: Oferece a opção de gerar e baixar a certidão em formato PDF.

Exibição de Dados no Navegador: Exibe os dados da certidão diretamente no navegador quando a opção de PDF não é selecionada.

Interface Responsiva: A interface é responsiva e se adapta a diferentes tamanhos de tela.

Validação de CNPJ: Valida o formato do CNPJ inserido.

Tratamento de Erros: Exibe mensagens de erro claras e informativas.

Contorno de CORS: Utiliza um proxy para contornar problemas de Cross-Origin Resource Sharing (CORS).

Tecnologias Utilizadas
HTML: Estrutura da página web.

CSS: Estilização da página e responsividade.

JavaScript: Lógica da aplicação, requisições à API e manipulação dos dados.

Arquivos do Projeto
index.html: Arquivo principal que contém a estrutura HTML, estilos CSS e código JavaScript.

Como Usar
Abra o arquivo index.html em um navegador web.

Insira o CNPJ desejado no campo "CNPJ".

Selecione a opção "SIM" para gerar e baixar a certidão em PDF ou "NÃO" para exibir os dados no navegador.

Clique no botão "Consultar".

Detalhes do Código
Estrutura HTML
O arquivo HTML contém a estrutura básica da página, incluindo:

Um formulário (#consultaForm) com campos para inserir o CNPJ (#cnpj) e selecionar a opção de PDF (#emitirPDF).

Um botão de consulta (<button type="submit">Consultar</button>).

Uma área para exibir os resultados (#resultado).

Estilos CSS
Os estilos CSS definem a aparência da página, incluindo:

Layout responsivo usando grid.

Cores e fontes personalizadas.

Estilos para o formulário e a área de resultados.

Código JavaScript
O código JavaScript contém a lógica principal da aplicação:

Função validarCNPJ(cnpj): Valida se o CNPJ inserido está no formato correto.

Função exibirErro(mensagem): Exibe mensagens de erro na área de resultados.

Função downloadPDF(url, cnpj): Realiza a requisição à API, trata a resposta e gera o arquivo PDF para download. Essa função lida com diferentes formatos de resposta (JSON e binário) e decodifica dados Base64, se necessário.

Função exibirResultado(data): Exibe os dados da certidão no navegador. Essa função também trata respostas que não são JSON e exibe o conteúdo original.

Evento submit do formulário: Captura o evento de envio do formulário, obtém os valores dos campos, realiza a requisição à API e chama as funções apropriadas para exibir os resultados ou gerar o PDF.

Contorno de CORS: Utiliza o proxy api.allorigins.win para contornar problemas de CORS ao realizar a requisição à API do TCU.

Tratamento de Timeout: Utiliza um AbortController para cancelar a requisição após um determinado período de tempo (10 segundos).

Funções Detalhadas
validarCNPJ(cnpj)
Valida se o CNPJ fornecido está no formato correto (14 dígitos numéricos).

javascript
function validarCNPJ(cnpj) {
    return /^\d{14}$/.test(cnpj);
}
exibirErro(mensagem)
Exibe uma mensagem de erro na área de resultados.

javascript
function exibirErro(mensagem) {
    const resultadoDiv = document.getElementById('resultado');
    resultadoDiv.innerHTML = `<div style="color: red; font-weight: bold;">${mensagem}</div>`;
}
downloadPDF(url, cnpj)
Realiza a requisição à API, trata a resposta e gera o arquivo PDF para download.

javascript
async function downloadPDF(url, cnpj) {
    try {
        const response = await fetch(url);

        if (!response.ok) {
            throw new Error(`Erro na requisição: ${response.status} ${response.statusText}`);
        }

        const contentType = response.headers.get('Content-Type');

        if (contentType && contentType.includes('application/json')) {
            // Dados em JSON (possivelmente Base64)
            const data = await response.json();
            let pdfBase64;

            if (data.certidaoPDF) {
                pdfBase64 = data.certidaoPDF; // Campo correto?
            } else if (data.contents) {
                try {
                    const jsonData = JSON.parse(data.contents);
                    pdfBase64 = jsonData.certidaoPDF; // Ajuste conforme a estrutura
                } catch (jsonError) {
                    throw new Error("Erro ao analisar JSON interno: " + jsonError.message);
                }
            } else {
                throw new Error("Formato de resposta desconhecido: " + JSON.stringify(data));
            }

            // Decodificar Base64
            const byteCharacters = atob(pdfBase64);
            const byteArrays = [];

            for (let offset = 0; offset < byteCharacters.length; offset += 512) {
                const slice = byteCharacters.slice(offset, offset + 512);
                const byteNumbers = new Array(slice.length);
                for (let i = 0; i < slice.length; i++) {
                    byteNumbers[i] = slice.charCodeAt(i);
                }
                const byteArray = new Uint8Array(byteNumbers);
                byteArrays.push(byteArray);
            }

            const blob = new Blob(byteArrays, { type: 'application/pdf' });
            const fileURL = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.href = fileURL;
            link.setAttribute('download', `certidao-tcu-${cnpj}.pdf`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            URL.revokeObjectURL(fileURL);

        } else {
            // Dados binários diretamente (ArrayBuffer)
            const buffer = await response.arrayBuffer();
            const blob = new Blob([buffer], { type: 'application/pdf' });
            const fileURL = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.href = fileURL;
            link.setAttribute('download', `certidao-tcu-${cnpj}.pdf`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            URL.revokeObjectURL(fileURL);
        }

    } catch (error) {
        exibirErro("Erro ao baixar PDF: " + error.message);
    }
}
exibirResultado(data)
Exibe os dados da certidão no navegador.

javascript
function exibirResultado(data) {
    const resultadoDiv = document.getElementById('resultado');
    resultadoDiv.innerHTML = '<h3>Resultado da Consulta:</h3>';
    
    if (data.contents) {
        try {
            const jsonData = JSON.parse(data.contents);
            resultadoDiv.innerHTML += `<pre>${JSON.stringify(jsonData, null, 2)}</pre>`;
        } catch (error) {
            resultadoDiv.innerHTML += `<p>Erro ao processar dados JSON: ${error.message}</p>`;
            resultadoDiv.innerHTML += `<p>Conteúdo original: ${data.contents}</p>`; // Exibe o conteúdo original
        }
    } else {
        resultadoDiv.innerHTML += `<p>Conteúdo não disponível para exibição direta.</p>`;
    }
}
Evento submit do formulário
Captura o evento de envio do formulário, obtém os valores dos campos, realiza a requisição à API e chama as funções apropriadas para exibir os resultados ou gerar o PDF.

javascript
document.getElementById('consultaForm').addEventListener('submit', async (e) => {
    e.preventDefault();

    const cnpj = document.getElementById('cnpj').value;
    const emitirPDF = document.getElementById('emitirPDF').value;

    if (!validarCNPJ(cnpj)) {
        exibirErro('CNPJ inválido. Deve conter 14 dígitos.');
        return;
    }

    try {
        const proxy = 'https://api.allorigins.win/get?url=';
        const url = `${proxy}${encodeURIComponent(
            `https://certidoes-apf.apps.tcu.gov.br/api/rest/publico/certidoes/${cnpj}?seEmitirPDF=${emitirPDF}`
        )}`;

        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), 10000); // Timeout de 10 segundos

        const response = await fetch(url, { signal: controller.signal });
        clearTimeout(timeoutId);

        if (!response.ok) {
            exibirErro(`Erro na requisição: ${response.status} ${response.statusText}`);
            return;
        }

        const contentType = response.headers.get('Content-Type');

        if (emitirPDF === 'true') {
            downloadPDF(url, cnpj);
        } else {
            if (contentType && contentType.includes('application/json')) {
                const data = await response.json();
                exibirResultado(data);
            } else {
                const text = await response.text();
                exibirErro(`Resposta não é JSON. Content-Type: ${contentType || 'desconhecido'}. Conteúdo: ${text}`);
            }
        }
    } catch (error) {
        exibirErro(`Erro: ${error.message}`);
    }
});
Configuração
Para executar este projeto, você precisará de:

Um navegador web moderno (Chrome, Firefox, Safari, etc.).

Acesso à internet para realizar as requisições à API do TCU.

Problemas Conhecidos
CORS: Devido à política de CORS, é necessário utilizar um proxy para realizar as requisições à API do TCU. O proxy utilizado neste projeto (api.allorigins.win) é público e pode ter limitações de uso. Em um ambiente de produção, é recomendado implementar um proxy próprio.

Formato da Resposta da API: A API do TCU pode retornar os dados em diferentes formatos (JSON, Base64, binário). O código tenta lidar com esses diferentes formatos, mas pode ser necessário ajustá-lo caso a API seja alterada.

Timeout: A API do TCU pode demorar para responder, resultando em um timeout. O código utiliza um AbortController para cancelar a requisição após 10 segundos, mas pode ser necessário ajustar esse valor.

Próximos Passos
Implementar um proxy próprio para evitar as limitações do proxy público.

Adicionar tratamento de erros mais detalhado e mensagens de erro mais informativas.

Implementar testes unitários para garantir a qualidade do código.

Adicionar mais opções de formatação para a exibição dos dados no navegador.

Permitir que o usuário configure o timeout da requisição.

Contribuição
Contribuições são bem-vindas! Sinta-se à vontade para abrir issues e pull requests.

Licença
Este projeto está licenciado sob a MIT License.
