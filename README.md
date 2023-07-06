# ProjectSR
import csv
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin

# Function to scrape product listings
def scrape_product_listings(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "html.parser")

    product_listings = []

    # Find all the product containers
    containers = soup.find_all("div", {"data-component-type": "s-search-result"})

    for container in containers:
        # Extract the product details
        product_url = container.find("a", class_="a-link-normal s-no-outline")["href"]
        product_name = container.find("span", class_="a-size-medium a-color-base a-text-normal").get_text(strip=True)
        product_price = container.find("span", class_="a-price-whole").get_text(strip=True)
        product_rating = container.find("span", class_="a-icon-alt").get_text(strip=True)
        product_reviews = container.find("span", class_="a-size-base").get_text(strip=True)

        # Append the product details to the list
        product_listings.append({
            "Product URL": product_url,
            "Product Name": product_name,
            "Product Price": product_price,
            "Rating": product_rating,
            "Number of Reviews": product_reviews
        })

    return product_listings


# Function to scrape product details
def scrape_product_details(url):
    response = requests.get(urljoin(base_url, url))
    soup = BeautifulSoup(response.content, "html.parser")

    product_details = {}

    # Extract the product details
    description = soup.find("div", {"id": "productDescription"})
    description_text = description.get_text(strip=True) if description else "N/A"

    asin = soup.find("th", string="ASIN")
    asin_value = asin.find_next_sibling("td").get_text(strip=True) if asin else "N/A"

    product_description = soup.find("h2", string="Product Description")
    product_description_text = product_description.find_next_sibling("div").get_text(strip=True) if product_description else "N/A"

    manufacturer = soup.find("th", string="Manufacturer")
    manufacturer_value = manufacturer.find_next_sibling("td").get_text(strip=True) if manufacturer else "N/A"

    # Store the product details in a dictionary
    product_details = {
        "Description": description_text,
        "ASIN": asin_value,
        "Product Description": product_description_text,
        "Manufacturer": manufacturer_value
    }

    return product_details


# URL for product listings
base_url = "https://www.amazon.in"
search_url = "https://www.amazon.in/s?k=bags&crid=2M096C61O4MLT&qid=1653308124&sprefix=ba%2Caps%2C283&ref=sr_pg_"
page_limit = 200  # Maximum number of pages to scrape
product_listings = []

# Scrape product listings from multiple pages until the desired number of URLs is reached
url_counter = 0
for page in range(1, page_limit+1):
    url = search_url + str(page)
    listings = scrape_product_listings(url)
    for product in listings:
        product_listings.append(product)
        url_counter += 1
        if url_counter == 200:
            break
    if url_counter == 200:
        break

# Scrape product details for each product URL
for product in product_listings:
    url = product["Product URL"]
    product_details = scrape_product_details(url)
    product.update(product_details)

# Export the scraped data to a CSV file
csv_filename = "scraped_data.csv"
fieldnames = ["Product URL", "Product Name", "Product Price", "Rating", "Number of Reviews",
              "Description", "ASIN", "Product Description", "Manufacturer"]

with open(csv_filename, "w", newline="", encoding="utf-8") as csv_file:
    writer = csv.DictWriter(csv_file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(product_listings)

print("Scraping completed and data exported to CSV.")

