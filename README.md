# WEB SCRAPING MERCADO LIBRE

Hola, este codigo que les voy a dejar a continuacion sirve para extraer todos los # Titualos, # Precios y la # URL del producto que necesitemos.
Solo debemos reemplazar la URL de la variable # siguiente.

import csv
import requests
from bs4 import BeautifulSoup
from lxml import etree
import re
import os

# Imprime el directorio donde se guarda el csv final
print(os.getcwd())

# URL inicial para hacer scraping
siguiente = 'https://listado.mercadolibre.com.ar/reloj-smartwatchh'

# Listas para almacenar los datos extraídos
lista_titulos = []
lista_url = []
lista_precios = []

# Bucle para iterar sobre todas las páginas de resultados
while True:
    # Realiza una solicitud HTTP GET a la URL
    r = requests.get(siguiente)
    # Verifica si la solicitud fue exitosa
    if r.status_code == 200:
        # Crea un objeto BeautifulSoup a partir del contenido de la respuesta
        soup = BeautifulSoup(r.content, 'html.parser')
        # Crea un objeto DOM a partir del contenido de la respuesta
        dom = etree.HTML(str(soup))
        # Busca todos los elementos h2 con la clase especificada y extrae el texto de cada uno
        titulos = soup.find_all('h2', attrs={"class": "ui-search-item__title shops__item-title"})
        # Reemplaza todas las comas por espacios en blanco en cada título
        titulos = [i.text.replace(',', ' ') for i in titulos]
        # Agrega los títulos a la lista de títulos
        lista_titulos.extend(titulos)
        # Busca todos los elementos a con la clase especificada y extrae el valor del atributo href de cada uno
        urls = soup.find_all('a', attrs={"class": "ui-search-item__group__element shops__items-group-details ui-search-link"})
        urls = [i.get('href') for i in urls]
        # Agrega las URLs a la lista de URLs
        lista_url.extend(urls)
        # Busca todos los elementos div con la clase especificada y extrae el texto de cada uno
        precio = soup.find_all('div', attrs={"class": "ui-search-price__second-line shops__price-second-line"})
        # Usa una expresión regular para extraer solo los dígitos y el punto decimal de cada texto de precio
        precio = [re.search(r'\d+\.\d+', i.text).group() if re.search(r'\d+\.\d+', i.text) else '' for i in precio]
        # Agrega los precios a la lista de precios
        lista_precios.extend(precio)

        # Busca el elemento span con la clase especificada y extrae el texto
        botoninicial = soup.find('span', attrs={"class", "andes-pagination__link"}).text
        # Convierte el texto a un número entero
        botoninicial = int(botoninicial)
        # Busca el elemento li con la clase especificada y extrae el texto
        can = soup.find('li', attrs={"class", "andes-pagination__page-count"})
        # Divide el texto en palabras y toma la segunda palabra (el número total de páginas)
        can = int(can.text.split(" ")[1])
    else:
        # Si la solicitud no fue exitosa, termina el bucle
        break

    # Si se ha llegado a la última página, termina el bucle
    if botoninicial == can:
        break

    # Usa XPath para encontrar el elemento a que representa el botón "Siguiente" y extrae el valor del atributo href
    siguiente = dom.xpath('//div[@class="ui-search-pagination shops__pagination-content"]//ul/li[contains(@class,"--next")]/a')[0].get('href')

# Abre un archivo CSV en modo escritura y codificación UTF-8
with open('datos_ventas.csv', 'w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file, quoting=csv.QUOTE_NONNUMERIC)  # Agrega el parámetro quoting
    # Escribe la fila de encabezados en el archivo CSV
    writer.writerow(['Titulo', 'Precio', 'URL'])

    # Itera sobre las listas de títulos, precios y URLs y escribe cada fila en el archivo CSV
    for titulo, precio, url in zip(lista_titulos, lista_precios, lista_url):
        writer.writerow([titulo, precio, url])
