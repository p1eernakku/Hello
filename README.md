import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin

url = "https://29.ru/text"

headers = {
    "User-Agent": "Mozilla/5.0"
}

try:
    response = requests.get(
        url,
        headers=headers,
        timeout=20
    )
    response.raise_for_status()
    soup = BeautifulSoup(response.text, "html.parser")
    articles = []
    seen_links = set()
    for link in soup.find_all("a", href=True):
        title = link.get_text(" ", strip=True)
        article_url = urljoin(url, link["href"])
        if (
            title
            and article_url.startswith("https://29.ru/text/")
            and article_url not in seen_links
        ):
            articles.append((title, article_url))
            seen_links.add(article_url)
        if len(articles) >= 100:
            break
    print(f"Найдено заголовков: {len(articles)}")
    print("-" * 60)
    for number, (title, article_url) in enumerate(
        articles, start=1
    ):
        print(f"{number}. {title}")
        print(f"   {article_url}\n")
    if len(articles) < 100:
        print(
            "Внимание: найдено меньше 100 заголовков. "
            "Возможно, сайт загружает материалы динамически."
        )

except requests.RequestException as error:
    print(f"Ошибка при загрузке сайта: {error}")
