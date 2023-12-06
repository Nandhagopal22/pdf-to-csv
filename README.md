# pdf-to-csv
collage project 
import PyPDF2 as pdf
import re
import json
from pydantic import BaseModel
import pandas as pd
addr_reg = re.compile(r'Addr')
header_reg = re.compile(r'TOTAL')
arr = []
dic = {}
header = {"h1": "LLT179", 
          "h2": "CARBANERVE 300 MG", 
          "h3": "10"}
def insert(dic, k, cus, sta, header = None):
    dic['customer'] = cus
    dic['h1'] = header['h1']
    dic['h2'] = header['h2']
    dic['h3'] = header['h3']
    dic['station'] = sta
    dic['bill_no'] = k[-9]
    dic['date'] = k[-8]
    dic['batch'] = k[-7]
    dic['MRP'] = k[-6]
    dic['exp_da'] = k[-5]
    dic['qty'] = k[-4]
    dic['free_qty'] = k[-3]
    dic['sales_value']= k[-2]
    arr.append(dic)
def header_insert(header, k):
    header['h1']= k[0]
    header['h2']= k[1]
    header['h3']= k[2]
def pdf_reader(page_text):
    global dic 
    global header
    for i, line in enumerate(page_text):
        if header_reg.match(line):
            k= page_text[i+2].split('   ')
            str_list = list(filter(None, k))
            if "GRAND" not in page_text[i+2] and "Page" not in page_text[i+2]:
                # print(str_list)
                header_insert(header, str_list)
        
        if addr_reg.match(line):
            k= re.sub(' +', ' ', page_text[i-1]).split(' ')
            dic= {}
            insert(dic, k, ' '.join(k[:-10]), k[-10], header)
            
            def next_pages(j):
                cur = re.sub(' +', ' ', page_text[j]).split(' ')
                if len(cur)> 8:
                    global dic
                    cus = dic['customer']
                    sta = dic['station']
                    global header
                    dic = {}
                    insert(dic, cur, cus, sta, header)
                    if '--' not in page_text[j+1]:
                        next_pages(j+1)

            next_pages(i)
if __name__ == "__main__":
    pdf_file = open("D:\Emman\Source\AGARWAL LIFECARE.pdf","rb")
    pdf_read = pdf.PdfFileReader(pdf_file)
    
    # page_info = pdf_read.pages[0]
    # page_text = page_info.extractText().split('\n')
    # pdf_reader(page_text)
    # # Extracting every pages
    for i in range(0, pdf_read.getNumPages()):
        page_info = pdf_read.pages[i]
        page_text = page_info.extractText().split('\n')
        pdf_reader(page_text)
    pdf_file.close()
df= pd.DataFrame.from_dict(arr)
df.to_csv("D:\Emman\Source\AGARWAL LIFECARE.csv")
print(json.dumps(arr, indent=2))
