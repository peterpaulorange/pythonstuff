import requests
from bs4 import BeautifulSoup
import re
import pandas as pd
name_price=dict()
name_photo=dict()
for page_num in range(1,97):
    Product_Name=[]
    Product_Price=[]
    URL="https://www.radioshack.eg/en/product/search&search=+&limit=100&page={}".format(page_num)
    page = requests.get(URL)
    soup = BeautifulSoup(page.content, 'html.parser')
    Names=soup.find_all('div',class_="name")
    for Name in Names:
        name=re.findall(r'<a href=.*>(.*)</a>',str(Name))
        name=name[0]
        Product_Name.append(name)
    Prices=soup.find_all('div',class_="price")
    for Price in Prices:
        price=re.findall(r'class="price">\s*(.*)<span class',str(Price))
        Product_Price.append(price[0])
    Images=soup.find_all('div',class_="image")
    for Image in Images:
        image=re.findall(r'src="(.*)" title="(.*)"/>',str(Image))
        try:
            image_url=image[0][0]
            image_title=image[0][1]
            name_photo[image_title]=image_url
        except:
            continue
    length=len(Product_Name)
    k=list(name_photo.keys())
    for i in range(length):
        name_price[Product_Name[i]]=Product_Price[i]
        if Product_Name[i] not in k:
            name_photo[Product_Name[i]]=""
name_photo={k: v for k, v in name_photo.items() if k in name_price.keys()}
name_price={k: v for k, v in sorted(name_price.items(), key=lambda item: item[0])}
name_photo={k: v for k, v in sorted(name_photo.items(), key=lambda item: item[0])}
Names=list(name_price.keys())
Prices=list(name_price.values())
Urls=list(name_photo.values())
print(len(Names))
print(len(Prices))
print(len(Urls))
raw_data={'Product Name':Names,'Price':Prices,'Photo URL':Urls}   
data_frame=pd.DataFrame(raw_data,columns=['Product Name', 'Price','Photo URL'])
data_frame.to_csv("RadioShack.csv",index = False)



-----////--
#!/usr/bin/python

import os, requests, re, shutil
import click
from requests_html import HTMLSession  # Cheat, helps to render pages for javascript


@click.command()
@click.option("--url")
def get_img(url):
    try:
        url = url.rstrip("/")
    except AttributeError:
        print("URL is not specified")
        exit()
    session = HTMLSession()
    r = session.get(url)
    content = str(r.content)  # Get all data
    relative_urls = re.findall(
        r"(?:[/|.|\w|\s|-])*\.(?:jpg|jpeg|gif|png)(?:[\w])*", content
    )  # I can parse raw html for images
    re_count = len(relative_urls)
    # print("\n".join(links))
    print("\n".join(relative_urls))
    print("Found " + str(re_count) + " images!")
    for img in relative_urls:
        download_img(url, img)
    print("DONE! Downloaded images: " + str(re_count))


def process_url(url, img_url):
    img_url = img_url.strip()
    result_url = img_url
    if img_url.startswith("http"):
        return result_url
    else:
        if img_url.startswith("//"):
            result_url = "https:" + img_url
            # print(result_url)
        else:
            if img_url.startswith("/"):
                result_url = url + img_url
            else:
                print("Invalid image URL!" + result_url)
    return result_url


def get_name(img_url):
    return str(re.findall(r"[^\/]+$", img_url)).strip("[']\"")


def download_img(url, img_url):
    result_img_url = process_url(url, img_url)
    response = requests.get(result_img_url, stream=True)
    if response.status_code == 200:
        os.makedirs(
            "images/", exist_ok=True
        )  # Create directory for images, skip if exists
        img_name = get_name(img_url)
        print("Downloading " + result_img_url)
        with open("images/" + img_name, "wb") as out_img:
            shutil.copyfileobj(response.raw, out_img)  # Save image
        del response
        print("OK")
    else:
        print(response.status_code + "\nFAILED TO DOWNLOAD IMAGE: " + result_img_url)


if __name__ == "__main__":
    get_img()
    
    -----///--
    #!/usr/bin/python

import os, requests, re, shutil
import click
from requests_html import HTMLSession  # Cheat, helps to render pages for javascript


@click.command()
@click.option("--url")
def get_img(url):
    try:
        url = url.rstrip("/")
    except AttributeError:
        print("URL is not specified")
        exit()
    session = HTMLSession()
    r = session.get(url)
    r.html.render()  # Attempt to bypass some scraping protection by rendering page for JS execution
    links = img_links(r.html)  # And i can use cheats =P
    link_count = len(links)
    # print("\n".join(links))
    print("Found " + str(link_count) + " images!")
    for img in links:
        download_img(url, img)
    print("DONE! Downloaded images: " + str(link_count))


# This function uses requests-html functionality to search for a tags
def img_links(html):
    def gen():
        for img_link in html.find("img"):
            try:
                src = img_link.attrs["src"].strip()
                print("SRC: " + src)
                if (
                    src.endswith("jpg")
                    or src.endswith("jpeg")
                    or src.endswith("png")
                    or src.endswith("gif")
                ):
                    yield src
            except requests.exceptions.MissingSchema:
                print("Invalid URL " + img_link)
                print("Terminating")

    return set(gen())


def process_url(url, img_url):
    img_url = img_url.strip()
    result_url = img_url
    if img_url.startswith("http"):
        return result_url
    else:
        if img_url.startswith("//"):
            result_url = "https:" + img_url
            # print(result_url)
        else:
            if img_url.startswith("/"):
                result_url = url + img_url
            else:
                print("Invalid image URL!" + result_url)
    return result_url


def get_name(img_url):
    return str(re.findall(r"[^\/]+$", img_url)).strip("[']\"")


def download_img(url, img_url):
    result_img_url = process_url(url, img_url)
    response = requests.get(result_img_url, stream=True)
    if response.status_code == 200:
        os.makedirs(
            "images/", exist_ok=True
        )  # Create directory for images, skip if exists
        img_name = get_name(img_url)
        print("Downloading " + result_img_url)
        with open("images/" + img_name, "wb") as out_img:
            shutil.copyfileobj(response.raw, out_img)  # Save image
        del response
        print("OK")
    else:
        print(response.status_code + "\nFAILED TO DOWNLOAD IMAGE: " + result_img_url)


if __name__ == "__main__":
    get_img()
