# TCC-Uso-das-Redes-Sociais
PyPDF2
scikit-learn
nltk
flask
requests

pip install -r requirements.txt

import os
import PyPDF2
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

# Função para ler PDFs e extrair texto
def ler_pdfs(caminho):
    textos = []
    for arquivo in os.listdir(caminho):
        if arquivo.endswith('.pdf'):
            with open(os.path.join(caminho, arquivo), 'rb') as f:
                leitor = PyPDF2.PdfReader(f)
                texto = ''
                for pagina in leitor.pages:
                    texto += pagina.extract_text()
                textos.append(texto)
    return textos

# Função para buscar informações
def buscar_informacao(textos, consulta):
    vetorizer = TfidfVectorizer()
    tfidf_matrix = vetorizer.fit_transform(textos + [consulta])
    similaridade = cosine_similarity(tfidf_matrix[-1], tfidf_matrix[:-1])
    indices = similaridade.argsort()[0][::-1]
    return [(textos[i], similaridade[0][i]) for i in indices]

@app.route('/upload', methods=['POST'])
def upload():
    caminho = 'inputs'
    textos = ler_pdfs(caminho)
    return jsonify({"mensagem": "PDFs carregados com sucesso!"})

@app.route('/buscar', methods=['POST'])
def buscar():
    dados = request.json
    consulta = dados['consulta']
    textos = ler_pdfs('inputs')
    resultados = buscar_informacao(textos, consulta)
    return jsonify(resultados)

@app.route('/analise_rede_social', methods=['POST'])
def analise_rede_social():
    dados = request.json
    url = dados['url']
    response = requests.get(url)
    # Aqui você pode adicionar lógica para analisar os dados da rede social
    return jsonify({"mensagem": "Dados da rede social analisados com sucesso!"})

if __name__ == '__main__':
    app.run(debug=True)

python app.py


{
  "consulta": "impacto das redes sociais na sociedade"
}


{
  "url": "https://api.exemplo.com/dados"
}
