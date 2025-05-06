# 📄 Extracción de Datos desde un PDF y Exportación a Excel con Python

A continuación se muestra un código en python para extraer ciertos datos de un pdf y transfórmalos a un Excel.
Para ello, es necesario realizar la instalación de las siguientes librerías:

```python
import pdfplumber
import re
import pandas as pd
````

Seguidamente, se realiza la extracción de todo el texto incluido en el pdf, para así poder clasificar en las diferentes listas creadas las
variables correspondientes según los caracteres clave.

```python
# Función para extraer información de un PDF
def extract_info_from_pdf(pdf_path):
    # Abrir el archivo PDF
    with pdfplumber.open(pdf_path) as pdf:
        all_text = ""
        # Extraer todo el texto de cada página
        for page in pdf.pages:
            all_text += page.extract_text()

    # Crear listas para cada tipo de dato que necesitamos extraer
    companies = []
    phones = []
    addresses = []
    urls = []
    emails = []
    descriptions = []

    # Expresiones regulares para cada tipo de dato
    phone_pattern = r'\+?\d{2,3}\s?\d{2,3}\s?\d{2,3}\s?\d{2,3}\s?\d{2,3}'
    email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b'
    url_pattern = r'https?://[^\s]+'
    company_address_pattern = r'(.+)\n(.+\d{5}\s+[^\n]+)'
    description_marker = r'(?<=CATEGORÍAS:).*'

    # Procesar el texto del PDF línea por línea
    lines = all_text.split("\n")
    for i, line in enumerate(lines):
        line = line.strip()

        # Buscar dirección y empresa juntas
        match = re.search(company_address_pattern, "\n".join(lines[i:i + 2]))
        if match:
            companies.append(match.group(1).strip())
            addresses.append(match.group(2).strip())
            continue

        # Buscar teléfonos
        if re.search(phone_pattern, line):
            phones.append(line)

        # Buscar correos electrónicos
        if re.search(email_pattern, line):
            emails.append(line)

        # Buscar URLs
        if re.search(url_pattern, line):
            urls.append(line)

        # Buscar descripciones
        if re.search(description_marker, line):
            descriptions.append(line.split(":")[1].strip())

    # Asegurar que todos los arrays tengan el mismo tamaño rellenando con None
    max_len = max(len(companies), len(phones), len(addresses), len(urls), len(emails), len(descriptions))
    for lst in [companies, phones, addresses, urls, emails, descriptions]:
        while len(lst) < max_len:
            lst.append(None)

    # Devolver todos los datos extraídos
    return {
        'companies': companies,
        'phones': phones,
        'addresses': addresses,
        'urls': urls,
        'emails': emails,
        'descriptions': descriptions
    }


# Ruta al archivo PDF
pdf_path = "nombre_archivo.pdf"

# Llamar a la función para extraer la información
extracted_data = extract_info_from_pdf(pdf_path)

# Crear un DataFrame de pandas para guardar los resultados
df = pd.DataFrame({
    'Company': extracted_data['companies'],
    'Phone': extracted_data['phones'],
    'Address': extracted_data['addresses'],
    'URL': extracted_data['urls'],
    'Email': extracted_data['emails'],
    'Description': extracted_data['descriptions']
})

# Ruta de salida para el archivo Excel
output_excel_path = "nombre_archivo.xlsx"
df.to_excel(output_excel_path, index=False)

print(f"Los datos han sido guardados en {output_excel_path}")
```



