# What is a spider/crawler
Our spiders are made up of two distinct pieces, crawling and parsing.

The crawling step is responsible for traversing the retailer website,
going through all relevant categories, clicking on "Next" links in listings pages and clicking through to individual product pages. Product URLs are then passed on to the parsing step.

The parsing step is responsible for parsing the content of a product
page and creating a **Garment** item populated with the **required fields**.

A complete spider is a python file containing two (or more) classes:

```python
class RetailerParseSpider(BaseParseSpider):
   ..
   ..

class RetailerCrawlSpider(BaseCrawlSpider):
  ..
  ..
```

A spider can have more classes if we are crawling across regions (for example, ASOS UK and ASOS AU), or if a number of retailers share the same page structure (for example, sites that belong to the Arcadia Group like Topshop, Topman and Dorothy Perkins).


# Crawler Specification
For each new spider, you will always receive a crawler specification with all the information you need to build the crawler.


## Retailer Types and Product Selections
We split types of products out into these groupings for crawling
- Apparel/Beauty/Non Apparel
- Homewares
- Unwanted Items

We only display apparel/beauty/non apparel items in the app. However we also want to crawl homeware items, but will hide these from the app. Please see below for details of what want/don't want for each grouping

We have tried to cover all possible categories you may come across, but this list is not exhaustive. If you see an item/category that you are not sure whether to crawl, please consult with EDITED. 

**Apparel**

Definition - A product made to be worn by males, females, girls or boys. This includes clothing, footwear, accessories, jewellery, bags, underwear, swimwear, nightwear.

Examples of apparel:
- Dresses
- Skirts
- Trousers
- Skirts
- Boots
- Rucksack
- Socks
- Earrings
- Sunglasses
- Suit Jacket
- Pyjamas
- Bikini

**Homewares**

Definition - A product made to furnish or accessorise a home/room.
Homewares should be marked with the industry “Homewares” flag. 

Homewares items we want to crawl:
- Lighting
- Furniture (Beds (incl mattresses)/tables/chairs/wardrobes/sofas etc)
- Soft Furnishings (curtains/bedding/towels/cushions/throws/table linen etc)
- Home Accessories (photo frames/candles/clocks etc)
- Rugs
- Fittings & fixtures (doorknobs/curtain poles)
- Storage (wardrobe/cabinets/chest of drawers etc)
- Seasonal items (christmas/halloween)
- Garden Furniture
- Decorative Accessories (ornaments/vases/book ends/mirrors etc)
- Bathroom Accessories (laundry bins/toothbrush holders)
- Bedding
- Cutlery/crockery (plates/glasses)
- Art
- Fabric
- Wallpaper
- Artificial flowers
- Doors
- Bins
- Curtains

You may find items under homewares sections on retailer sites that we do not want to crawl

Homewares items we do not want to crawl:
- Carpets/vinyl flooring
- Paint
- Hardware (Screws/lightbulbs)
- Cookware (pots/pans/knives/kitchen gadgets/utensils) 
- Bakeware (cupcake tins/cases/piping bags/cake stand)
- Bar accessories
- Stationery (greeting cards/wrapping paper/photo albums/calendars)
- Fittings and fixtures(bathroom cabinets/fitted kitchens/taps/baths) 
- Fireplaces
- Ironing boards
- BBQ
- Hardware
- Pet care
- White goods/cookers/microwave
- Hot tub
- Irons
- Vacuum cleaner
- Cleaning Products
- Real plants

**Unwanted items**

Definition - An item which does not fit our criteria as being part of the apparel or homewares industry. 

These items should not be included in the crawl:
- Tech items (computers/headphones etc)
- Toys/Games
- Sports equipment
- Electrical appliances
- White Goods
- Gift cards
- Food & drink
- Flowers
- Books/DVD/CD
- Health Items (bandages/wigs/medicine)
- Haberdashery
- Pet products
- Baby safety/accessories/prams/car seats

**Beauty**

Beauty items we want to crawl:
- make up
- body wash
- face creams/wash
- moisturiser
- perfume
- manual tools (nail files/clippers/make-up brushes)
- hairbrush
- hair styling products

Beauty items we do not want to crawl:
- electrical beauty items (hair dryers/hair stylers/foot spa/electrical massagers/stomach toners)
- supplements
- health food
- medical supplies
- medical implements
- medicine
- first aid
- wheelchairs
- contraception
- sanitary items
- contact lenses
- razors/hair removal

**Non Apparel sold alongside apparel items**

Definition - A non apparel item which has importance to the scope of the apparel selection. It is sold alongside apparel products.

We want to crawl:
- Books
- Novelty Gifts
- Certain stationary

Example sections:
http://www.asos.com/women/gifts-for-her/cat/pgecategory.aspx?cid=16095&wt.ac=ww%7cnav%7cgifts&via=top
http://www.urbanoutfitters.com/uk/catalog/category.jsp?id=FUN-GAMES-EU&cm_sp=HOME-_-L2-_-HOME:FUN-GAMES-EU
http://www.topshop.com/en/tsuk/category/bags-accessories-1702216/gifts-novelty-837/N-7v8Zdgl?No=0&Nrpp=20&siteId=%2F12556


**Note**: Only enable crawling homeware products if specified and always set the `industry` flag to homeware.

**Note**: If ever in doubt about what should and should not be crawled, just ask!


When we specify a **Retailer Type**, we will specify **multi** (e.g. department stores like John Lewis), **apparel** (e.g. ASOS) or **homeware** - if the retailer is an apparel retailer, or homeware retailer, please pass all products through the pipeline whether they match these categories or not.

If the retailer is a **multi** retailer, you should take great care to only pick up products which fit into the defined categories below (apparel - including beauty, or homewares) - and not pick up and send products to the parse spider that are not clothing/accessories/beauty products (for example toys/cutlery/beds/sofas etc should not to be picked up)! This can be done by either using XPaths that omit these categories or using `denied_paths` to filter out products from certain categories or using `allowed_paths` to only allow products from certain URL schemes.

Please make this logic obvious, because in future, we will want to ingest these kinds of products.

Note that the above will change in the future as we develop the product. You will be updated when this happens. 

#Crawler Output

## Flash Sales

We are also interested in crawling flash sales retailers, which are retailers that have timed sales such as [Gilt](http://www.gilt.com/) or [Rue La La](https://www.ruelala.com). If building a new spider for a flash sales retailer (this will be explicitly mentioned to you), then you should go ahead building the spider as per normal but with four important things to keep in mind:
- The `ParseSpider` should have `flash_sales = True` set
- Add a dict `'flash_sale'` to the garment, which will be of the following format:
```python
"flash_sale": {
    "retailer_sale_id": "a-unique-retailer-sale-id",
    "flash_sale_id": "a-unique-flash-sale-id",
    "name": "Labor Day Sale",
    "url": "http://gilt.com/sale/women/labor-day-sale",
    "starts_at": 1412697600,
    "ends_in": 1412827200
}
```
More specifically:
* `retailer_sale_id` is the retailer's id for the flash sale
* `flash_sale_id` is a combination of the `retailer_sale_id` and the retailer (hashed)
* `url` should be the base flash sale URL
* `name` should be the name given to the flash sale which the product belongs to
* `ends_in` is not meant to be parsed. (However, it should attempt to be of the format: 'x days, y hours' or '20 mins'.)
* `starts_at` and `ends_at` should be timestamps
*Note*: If the exact end time is known, then `ends_at` should be used, which is the date time in ISO format
- Flash sales sometimes show the number of products remaining per SKU, so if this information is available we should store this in the `sku` dict field, per SKU, in the field `stock_level`.
- Flash sales will most certainly show a previous price, original price or an RRP price - this should always be picked up as a `previous_price`.
- A Flash sale __must end__ ([proof?](http://en.wikipedia.org/wiki/Deal_of_the_day)). If the flash sale details on the retailer site do not have an ending date then __do not add a `flash_sale` dict to the crawl document__.  I.e. We do not want flash sales which do not have an end date (`ends_at` or `ends_in` must be present).

The [`gilt-us`](https://github.com/EDITD/skuscraper/blob/310e28ab4e2ebbb05f66bda560af83f428a580f6/spiders/gilt_spider.py#L133-L156) spider would be a good example, where you can see how we go about extracting information for a product's `flash_sale` dict.

[1] [skuscraper/items.py](items.py) contains the definition for the what a Garment could contain.

# Development and Test Environment

At the moment we use an old version of Scrapy (`v0.18.4`); we do intend to upgrade in the future and we will let you know when this happens.

There is an Ansible-provisioned Vagrant [1] environment to set up the whole
environment which is needed to crawl/parse a retailer. This is
located in the [skuscraper/playbook](playbook) directory. This environment is
set up by issuing

```
vagrant up
```

in the `skuscraper` directory and is accessible by issuing

```
vagrant ssh
```

once the machine is provisioned and ready.

Once in the environment one can test crawl a retailer by issuing

```
scrapy crawl retailername-crawl
```

To test parse an item

```
scrapy parse --spider=retailer-parse url
```

For example

```
scrapy parse --spider=newlook-parse
http://www.newlook.com/shop/womens/dresses/parisian-black-aztec-side-panel-bodycon-dress_294642077
```

[1] To use vagrant (at date of writing) you need the following dependencies:

```
VirtualBox 4.2.16
Vagrant version 1.6.3
```

# Crawling

The crawl bit of a spider, as mentioned previously, is responsible for
traversing a retailer and might look something like this:

```python
# BaseParseSpider is a class located in base.py that contains some
# methods used across all spiders
class RetailerCrawlSpider(BaseCrawlSpider):
    # the name field names the crawl spider so that scrapy knows which
    # spider to use when issuing a "scrapy crawl retailer"
    name            =  "retailer-crawl"
    # The market field specifies the market for the spider, for instance
    # "US", "UK", "DE"
    market          =  "US"
    # The retailer field specifies the retailer name in lowercase
    retailer        =  "retailer"
    # The allowed_domains field specifies which domains and sub-domains
    # a spider is allowed to crawl
    # http://doc.scrapy.org/en/0.14/topics/spiders.html#scrapy.spider.BaseSpider.allowed_domains
    allowed_domains = ["retailer.com", "product.retailer.com"]
    # The URL(s) the spider should start crawling from
    # http://doc.scrapy.org/en/0.14/topics/spiders.html#scrapy.spider.BaseSpider.start_urls
    start_urls      = ["http://retailer.com/"]

    def __init__(self):
        super(RetailerCrawlSpider, self).__init__()
        # The parser the spider should use when encountering a product page;
        # this is used along with self.parse_item
        self.parse_spider = RetailerParseSpider()

    # A dictionary which (usually) contains two keys, namely "listings" and
    # "products" which are used by the rules
    interesting_xpaths = {
        # The listings key contains a list of XPaths used to traverse the
        # website. These rules might contain XPaths for how to get to different
        # subcategories and how to locate a "next page" link for a subcategory
        # etc. When encountering these the crawl spider will continue crawling
        # these urls
        "listings": [
          '//ul[@class="all-different-categories-from-dropdown"]',
          '//ul[@class="next-page"]'
        ],
        # The products key contains a list of XPaths used to pick up
        # product URLs that will be sent to the parse spider
        "products": [
          '//ul[@class="container-of-all-garments"]'
        ],
    }

    # `allowed_paths` contains a list of regular expressions for paths that are
    # allowed to be crawled/parsed - this is often not used.
    allowed_paths = []
    # `denied_paths` contains a list of regular expressions for paths that are
    # disallowed to follow/crawl, usually used to filter out unwanted categories
    denied_paths = []

    # `rules` specifies rules for how the spider will crawl/parse
    # the website using the XPaths provided
    # http://doc.scrapy.org/en/0.14/topics/spiders.html#scrapy.contrib.spiders.CrawlSpider.rules
    # http://doc.scrapy.org/en/0.14/topics/spiders.html#crawling-rules
    rules = (
             Rule(
                  SgmlLinkExtractor(restrict_xpaths=interesting_xpaths["listings"],
                        allow=allowed_paths, deny=denied_paths),
                  process_links="process_links",
                  # `process_links` is specified in the `BaseCrawlSpider` and all it does
                  # is to ensure that we are not crawling the same URLs multiple times. Finally,
                  # this uses `self` to crawl the URL provided.
                  ),
             Rule(
                  SgmlLinkExtractor(restrict_xpaths=interesting_xpaths["products"],
                        allow=allowed_paths, deny=denied_paths),
                  callback="parse_item"
                  # `parse_item` is specified in the `BaseCrawlSpider` (sometimes this
                  # has to be overridden when retailer specific manipulation of the picked
                  # up URL needs to be done). All it does is call self.parse_spider.parse
                  ),
             )
```

# Parsing

```python
# `BaseParseSpider` is a class located in base that contains some helper
# functions that are used across most spiders.
class BCBGParseSpider(BaseParseSpider):
  # The name field names the parse spider so that scrapy knows which spider to
  # use when issuing a "scrapy parse --retailer=retailer-parse URL"
  name            =  'retailer-parse'
  # The retailer field specifies the retailer name, always in lowercase
  retailer        =  'retailer'
  # The `allowed_domains` field specifies which domains and sub domains
  # the spider is allowed to parse
  allowed_domains = ['retailer.com']
  # The market field specifies the market for the spider, e.g. "US" or "UK"
  market          =  'US'

  # The parse method is the entry point for when the crawler has found a url
  # to be parsed.
  def parse(self, response):
    # HtmlXPathSelector takes a response.body (html from the requested URL)
    # and parses it to enable us to use XPaths to extract data.
    # http://doc.scrapy.org/en/0.14/topics/selectors.html#scrapy.selector.HtmlXPathSelector
    hxs = HtmlXPathSelector(response)

    # The garment is the item that we will populate with relevant data, see
    # https://github.com/EDITD/docs/wiki/Spiders#items for more information.
    garment = Garment()
    # The sku_id is the unique id the retailer has given the item on their
    # webpage, this is used to construct a product_hash.
    sku_id = self.product_id(hxs)

    # self.duplicate makes sure that we don't parse the same item twice in a
    # crawl, if the item is present in different listings pages for instance.
    # If it's the first time we see the item it populates garment['retailer_sku']
    # with the sku_id, if we have seen the garment before this method returns
    # `True` and the parse of the item is cancelled.
    if self.duplicate(garment, sku_id):
      return

    # out_of_stock checks for visible signs (defined by the spider author) to
    # determine if the item is out of stock on the website. If it is we only
    # need to return a subset of all fields since some fields are not interesting
    # in case of out of stock.
    if self.out_of_stock(hxs, response):
      # The out_of_stock_garment is a garment that contains a subset of the
      # fields of a full garment.
      return self.out_of_stock_garment(hxs, response)

    # Note that the following clause could be replaces with
    # self.boilerplate_normal(garment, hxs, response) from
    # BaseParseSpider which will do the same thing.
    #
    garment['uuid']         = response.meta.get('uuid')
    garment['date']         = self.utc_now()
    garment['url']          = response.url
    garment['market']       = self.market
    garment['retailer']     = self.retailer
    garment['product_hash'] = self.product_hash(garment['retailer']+'_'+garment['retailer_sku'])
    garment['brand']        = "retailer"
    garment['name']         = self.product_name(hxs)
    garment['description']  = self.product_description(hxs)
    garment['care']         = self.product_care(hxs)


    garment['skus']         = self.skus(hxs)
    garment['image_urls']   = self.image_urls(hxs)
    garment['gender']       = 'women'

    # Here we return the populated garment to be inspected by the pipelines,
    # see https://github.com/EDITD/docs/wiki/Spiders#pipielines
    return garment

  def out_of_stock(self, hxs, response):
    # This function just does a simple check to see if the retailer has
    # marked the garment as out of stock.
    #
    if url_query_parameter(response.url, 'oos') == 'true':
      return True

  def out_of_stock_garment(self, hxs, response):
      """Return an out-of-stock garment."""
      garment = Garment()
      # these are the required fields for an out-of-stock garment.
      garment['out_of_stock'] = True
      garment['date']         = self.utc_now()
      garment['url']          = response.url
      garment['retailer']     = self.retailer
      garment['retailer_sku'] = self.product_id(response.url)
      garment['product_hash'] = self.product_hash(garment['retailer']+'_'+garment['retailer_sku'])
      garment['crawl_id']     = self.crawl_id
      return garment

  def product_id(self, hxs):
      # The product_id is the unique id the retailer has assigned the
      # garment.
      #
      product_id = hxs.select('//span[@itemprop="productID"]/text()').extract()[0].strip()
      return product_id

  def product_name(self, hxs):
      # The product name is the name of the product.
      #
      return titlecase(hxs.select('//h1[(@class="product-name")]/text()')[0].extract_unquoted())

  def product_description(self, hxs):
    # Picks up the description for the garment.
    # sanitize is defined in the BaseParseSpider and sanitizes a list
    # of strings or a string, removing trailing spaces and unicode
    # whitespace and similar.
    #
    desc = hxs.select('//div[(@itemprop="description")]//text()').extract()
    return self.sanitize(desc)

  def product_care(self, hxs):
      # Picks up the care, i.e washing instructions, materials the
      # garment was made from and similar.
      #
      care_list = hxs.select('//div[(@id="productBulletPoints")]/ul/li/text()').extract_unquoted()[1:]
      return [care.strip() for care in care_list]

  def prices(self, hxs):
    # Helper function that is called in self.skus. Picks up price,
    # previous price (if present) and currency for the garment.
    # CurrencyParser.lowest_price picks up the lowest price from the
    # supplied string and converts it into a integer, example
    # "$99.95" -> 9995
    # CurrencyParser.currency takes a string containing the currency
    # and returns a string representation of it, example
    # "$99.95" -> "USD"

    price, previous_price, currency = '20.00', '25.00', 'USD' # Just some dummy data
    prices = hxs.select('//div[@id="product-content"]/div[@class="product-price"]/span/text()').extract()
    if len(prices) > 1:
      _, previous_price, price = prices
    else:
      price = prices[0]
    return CurrencyParser.lowest_price(price),\
           CurrencyParser.lowest_price(previous_price),\
           CurrencyParser.currency(price)

  def skus(self, hxs):
      # Skus will be a dictionary containing all the different
      # colourway/size combinations. Example
      # {
      #  'bluelarge' :{
      #    'size': 'Large',
      #    'colour': 'Blue',
      #    'price': 9995,
      #    'currency': 'USD',
      #  },
      #  'bluemedium' :{
      #    'size': 'Medium',
      #    'colour': 'Blue',
      #    'price': 9995,
      #    'currency': 'USD',
      #    'out_of_stock': True
      #  }
      # }
      # Not that the id of the size/colour can be whatewher you want,
      # but its essential that it will be match-up-able if the
      # retailer changes the website.

      skus = {}

      colour = titlecase(hxs.select('//span[@class="selected-value"]/text()').extract()[0]).strip()
      sizes = hxs.select('//div[@class="val_div"]//select[@id="va-size"]/option')[1:] #First option is blank

      price, previous_price, currency = self.prices(hxs)

      for size_cont in sizes:
        size = self.sanitize(size_cont.select('text()').extract()[0])
        out_of_stock = bool(size_cont.select('@style').extract()) #Out of stock items have style=color:#BBB;
        skus[size] = sku = {}
        sku['colour'] = colour
        sku['size'] = size
        sku['price'] = price
        sku['currency'] = currency
        if previous_price:
          sku['previous_price'] = previous_price
        if out_of_stock:
          sku['out_of_stock'] = True
      return skus

  def image_urls(self, hxs):
    # This function returns a list of urls for all the images that
    # belong tho this garment.
    return hxs.select('//div[@class="thumb"]/a/@href').extract()

```


# Items
http://doc.scrapy.org/en/latest/topics/items.html

We have defined an item called Garment containing the following
fields.

```python
class Garment(Item):

    # This field is used to carry over information when the parsing of
    # an item requires the sending of multiple requests.
    #
    meta = Field(default='')

    # A product hash is a 24 char sha1 of retailer + retailer_sku
    # Should be a string.
    #
    product_hash = Field()

    # Utc epoch timestamp
    # Should be an int.
    date = Field()

    # Not used and set to None
    # TODO make absolutely sure not in use and remove
    #
    uuid = Field()

    # Will be retailer-crawl
    # I don’t think this is used,
    # TODO make absolutely sure not in use and remove
    #
    spider_name = Field()

    # Not in use
    # TODO make absolutely sure not in use and remove
    #
    crawl_date = Field()

    # now = datetime.utcnow()
    # ymd       = now.strftime("%Y%m%d")
    # epoch     = str(int(time.mktime(now.timetuple())))
    # gibberish = "".join([chr(int(random() * 26) + 97) for x in range(4)])
    # crawl_id = "-".join((self.name, ymd, epoch, gibberish))
    #
    # Pretty sure not in use
    # TODO make absolutely sure not in use and remove
    #
    crawl_id   = Field()


    # This will be response.url
    # Should be a string
    #
    url = Field() # normalized url.

    # Not in use
    # TODO make absolutely sure not in use and remove
    #
    urls = Field()

    # This is the same as garment['url'].
    # TODO make absolutely sure not in use and remove
    #
    url_original = Field()

    # A link to the facebook like url, used to extract facebook like
    # information for a product
    # Should be a string
    #
    facebook_like_url = Field()

    # Lowercased, no spaced, no alphanumerical string of the retailer
    # name
    # should be a string
    #
    retailer = Field()

    # A two uppercase lettered representation of the country code for
    # the crawled Garments market. "DE", "US", "UK" for example
    # Should be a string
    #
    market = Field()

    # The name of the product
    # Should be a string
    #
    name = Field()

    # The description of the product
    # Should be a string or a list of strings
    description = Field()

    # The care information of a product if present.
    # For instance "50% cotton, 50% chemical waste. Wash in 40 degrees"
    # Should be a string or a list of strings
    #
    care = Field()

    # The category the product recides in, i.e "T-shits", "Dresses"
    # Should be a string
    #
    category = Field()

    # The category bredcrumb texts. I.e
    # ["Mens", "Sale", "T-shirts"]
    # Should be a list of strings
    #
    categories = Field()

    # Only used in special cases with regards to gender extraction.
    # This field will be a full trail of all urls that have been
    # visited since the crawl started to when the product was found.
    # I.e
    # ['http://homepage.com', 'http://homepage.com/mens',
    # 'http://homepage.com/mens/tshirts',
    # 'http://homepage.com/product?100']
    # Should be a list of strings
    #
    trail = Field()

    gender = Field()

    # The brand of the parsed item.
    # Should be a string
    #
    brand = Field()

    # The unique identifier that the retailer uses to identify the
    # product
    # Likely to be a something like "317876" or "GY-774"
    # Should be a string
    #
    retailer_sku = Field()

    # A field populated by the previous_price of a product
    # Not in use anymore.
    # TODO make absolutely sure not in use and remove
    #
    previous_price = Field()

    # A field populated by the NormalisePricesPipeline.
    # I will be the lowest price of all the sku-prices
    # Should be a int
    #
    price = Field()

    # A field populated by the NormalisePricesPipeline.
    # Will be the first encountered currency from the sku-currencies.
    # Should be a string
    #
    currency = Field()

    # A list containing urls for all the images for a product.
    # Should be a list of strings
    #
    image_urls = Field()

    # Not in use
    # TODO make absolutely sure not in use and remove
    #
    image_paths = Field()

    # A field that is used when a product requiers multiple requests
    # to get all the sku information for all different colours.
    # This should probably live in the meta field, and also should not
    # be used for new spiders since we should regard different colours
    # of the same garment as different products.
    # Should be a list of Request objects.
    #
    sku_urls = Field()

    # A dict looking like so
    # { 'sku_id': {
    #               'colour': 'the colour of the sku',
    #               'price': 'the current price of the sku',
    #               'previous_price': 'the previous price if present',
    #               'currency': 'the currency of the sku',
    #               'out_of_stock': 'true if the sku is sold out',
    #             },
    #   'sku_id2': ...
    # }
    #
    # sku_id should be a string
    # colour should be a string
    # price should be a int
    # previous_price should be a int (if present)
    # currency should be a string
    # out_of_stock should be a bool, if present, otherwise leave
    # unset.
    #
    skus = Field()

    # Not in use anymore
    # TODO make absolutely sure not in use and remove
    #
    recommendations = Field()

    # Not in use anymore
    # TODO make absolutely sure not in use and remove
    #
    review_score = Field()

    # Not in use anymore
    # TODO make absolutely sure not in use and remove
    #
    number_of_reviews = Field()

    # Boolean field that marks if a entire product(all its skus) are
    # sold out
    # Should be a bool if the product is out of stock, otherwise leave
    # unset
    #
    out_of_stock = Field()

    # A field that marks if a garment was picked up in a
    # outlet section of a site. Note that outlet is not the same as a
    # sales item
    # Should be a bool if the product is a outlet product, otherwise
    # leave unset
    #
    outlet = Field()

    # A dict for storing which flash sale the product is in.
    flash_sale = Field()
```

## Creating garments with or without duplicates

At the beginning of `parse` methods, the `Garment` item to be returned is
usually created like this:

```python
    garment = Garment()
    if self.duplicate(garment, retailer_sku_id):
        return
```

or

```python
    garment = self.new_unique_garment(retailer_sku_id)
    if not garment:
        return
```

(While there are two different methods, they both check the same thing.
It is a known `TODO` to refactor one into the other :))

What these do is simply check whether we have already seen that specific
`retailer_sku_id` before in the crawl.
(For example, a retailer having the same product available at two different
urls.)

So to avoid generating another crawl document for a product that we have
already seen, we use one of the duplicate checking methods above.

However, in rare circumstances, we want to __allow__ duplicates, i.e. crawling
the same product more than once within the same crawl.

This is mainly when the retailer will have the product listed twice on their
site, but have slightly different information at each URL.

For example, they might market an item for both men and women, and list it at
two urls (one would say 'men' and one 'women').
In this case, if we maintained the duplicate check, we would only pick up one
of the 'men' or 'women' genders. (Which will be the first one crawled, as the
next one will be skipped due to the duplicate check.)

But we want to record the fact that the retailer listed the product twice.
In these instances, it is fine to __not__ do the duplicate check, and instead
instantiate the garment using:

```python
    garment = self.new_garment(retailer_sku_id)
```

So we will generate two crawl documents for that product (with the same id).
These two crawl documents will get merged in a later process, so all the
crawler needs to do is generate the crawl documents.

Please check with us if you feel that a crawler benefits from allowing
multiple crawls of the same product! When we need you to ignore the duplicate
check, we will let you know via a Github issue.


## Merchandised Info

Merchandised information is information on the way the product is being sold; this is usually and offer or status. Any merchandising information about the product should be stored in a key called `merch_info`, which is a list.

This information includes:

* Any multi-buy deal the product may be in.
   - "Buy One and Get One Free"
   - "3 for the price of 2 on all outerwear"
   - "Cheapest item free"
* Any special status
   - "Pre-Order available"
   - "Online Only"
   - "Catwalk Exclusive"
   - "Exclusive to (retailer)"
* Any percentage discount offer
   - “20% off dresses for a limited time”
   - “Spend £100, get 10% off”
* Delivery offers
   - “Free delivery when you spend £100 or more”
   - “Delivery only £1, this week only”
* Free Gift offers
   - “Receive a sample of perfume with any order”
   - “Free tote bag included with any jeans purchase”

The goal is to capture the information and it will be processed later.
So there is no need for the crawler code to try and parse the text. It should
simply add the exact string to `merch_info` __as it appears on the website__.

E.g.

```json
    "merch_info": ["Buy one and get one free!"]
```

Add multiple items the same way.

```json
    "merch_info": ["Pre-Order available", "3 for 2 on all outerwear"]
```

To decide what merchandising offers we want to capture:

* We only want offers that are linked to specific items. Offers could be site wide or only for certain categories, but we don't want to pick information from banners or homepages and apply to all products as we can't be sure they are included. We only pick merchandising information from the product page.

* If there is a sales category page (e.g. http://www.topshop.com/en/tsuk/category/sale-offers-4181277?geoip=noredirect
  ) then this will most likely be in the category trail (so doesn't need to be
  in the `merch_info` key)

* Remove merchandising information from product descriptions. No data should be repeated between data points.

* Always capture relevant merchandising info offers (even if the pricing
  information was collected as part of merchandising discount).
  So if there was price reduction, capture the pricing info, but also capture
  any text about this offer (if it is relevant merchandising info).
  I.e. If a discount has been captured, and there is still merchandising info,
  capture that too!

In terms of making extra requests, we should consider the product page to
start below the navigation and banners at the top of the page,
and only extract information from the relevant section.

So if there is a visible element 'within' the product section, and it is
specific to the product (but it was fetched via an ajax call), then we should
make an extra request to fetch offer information. Otherwise not.

A good rule of thumb would be to ask whether the offer could be (or is!)
displayed on every other product on the site.

If so, it is not specific to this product, and shouldn't be included.

If the information is specific only to one sku, then store it against the
sku.

e.g.
```json
"skus": {
    "10001": {"size": ..., "merch_info": "Pre-Order available"},
    "10002": {"size": ...}
}
```


## Spider names

The convention is to have a parse spider and a crawl spider for each retailer.

The parse spider is called `${retailer}-parse` and the crawl spider is called `${retailer}-crawl`.

(Most of the time this is achieved via mixins. E.g.: https://github.com/EDITD/skuscraper/blob/70ee087b49e73660ec80e503663adb5b62eb8bd0/skuscraper/spiders/marcjacobs_spider.py#L221-L222 )

As for the value for the retailer name, please ensure that the retailer name also contains the region of the crawler.

So we have

```pythonthon
class Mixin(object):
    retailer = "gilt-us"
    market = "US"
    ...
```

instead of

```pythonthon
class Mixin(object):
    retailer = "gilt"
    market = "US"
    ...
```

Even if a retailer only exists in one market, it is important to specify the region in the retailer value. (After all, it exists in only one market now, but that may change in the future.)

It is important to get the value for `retailer` correct because once chosen __it is not possible to change it!__.

(This is why you see some crawlers that do not have the region in the `retailer` value. These ones were chosen before we settled on this convention, and now sadly it is too late to change.)

If in doubt: please ask!

## Prices and SKUs

It is entirely expected that each sku in a product may have a different price.

As a result, we have several different price values for the product in general. E.g. the lowest price from all the skus, the highest ever seen price etc etc.

Luckily for crawler developers, they need not worry about any of this! It is handled later on by a separate process that occurs after the documents have been crawled.

So the crawler does not have to decide on a price for the product. It simply needs to ensure that the price is recorded for each sku.

Note that each sku needs to have a `price` attribute. It is fine if each sku has the same `price`.

One final note: please ensure that the values of the `price` and `previous_price` are integers in the relevant 'pence' or 'cent' amounts.

E.g. if a sku is priced at "£55" then we need for the sku:

```json
    "price": 5500,
    "currency": "GBP",
```

So the price __has__ to be an int, and __has__ to be multiplied by 100 (i.e. the pence or cent amount).

This is so that we can operate on integers and not have to worry (too much!) about floating point issues.


## Item Gender

The gender field is a string and must be one of the following

* men
* women
* girls
* boys
* unisex-kids
* unisex-adults

When determining which gender value to use, it is useful to consider whether
the garmet is for adults or for children, and also whether it is for
males, females, or suitable for both.

|              |  Male  |   Female   |     For both    |
|--------------|--------|------------|-----------------|
| __Adults__   | men    | women      | unisex-adults   |
| __Children__ | boys   | girls      | unisex-kids     |

If unable to determine a suitable value, please use unisex-adults.

## SKU price (and previous price)

Sometimes the retailer will also list the price that the item __used__ to be sold at.

E.g. Was ~~$29.99~~, Now $19.99

In this case, we should record the `price` as 1999, but store the initial price as `previous_price`.

E.g.

```json
    ...,
    "skus": {
        "red-large": {
            "color": "red",
            "size": "large",
            "price": 1999,
            "previous_price": 2999
        }
    },
    ...
```

(Note that each sku within a product could have a different `price`, and a different `previous_price`.)

Sometimes a retailer will have multiple levels of sales.
E.g. "10% off, and a further 50% off".

In this case, they often have multiple 'strike-through' prices.

E.g. Regular ~~$100~~, Sale ~~$90~~, Now ~~$45~~

At the moment, we have no mechanism for storing the intermediate sale.

So if this is encountered, then we take the highest 'strike-through' price as the `previous_price`.
(And of course, the price at which the item is currently selling is `price`.)

So in the above example:

```json
            "price": 4500,
            "previous_price": 10000
```

# Language

We are able to crawl sites in any language, and currently use this information to give users the option to display product information in the original crawled language. The default language for the app is English, we translate the name/description to English and store the original language. 

Translation does not have to be done by the crawler. This is handled later on in a separate, post-crawl process.

Always aim to crawl the site in the language of the region that we are crawling (where available). This allows us to train our internal classifiers to recognise a number of languages. 

We will indicate the language/ISO code to use when building an issue.

All the crawler needs to do is ensure a crawl document gets translated is to give the crawl document a `lang` field.

The value of the `lang` field is a string and must be the two letter iso-code for the __original__ language. (Please make it lower case!)

* [http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)

So if a German site was being crawled, the crawler would ensure that each crawl document had:

```json
    "lang": "de"
```


# Merchandised Discounts

It is important that we capture all types of discounts offered by a retailer on their site. At present, most spiders capture a `previous_price` for every SKU if one can be found. We want to take that approach and expand it to cater for more scenarios.

From this point onwards, we want to replace `previous_price` with `previous_prices` which should be a list of previous prices, even if there is just one. If there are no previous prices, then this should be an empty list or no value assigned to `previous_prices`.

## Price Reductions
Price reductions must be stored on a per SKU basis.

If there are previous prices for the entire product, they should also be stored as a list of `previous_prices` at the product level (irrespective of the SKU `previous_prices`).

### Reduction ("Was $y, now $x")
A simple case when the price of a SKU has been reduced; all that is needed is to record a `previous_prices` for each SKU.

```json
{
    "Black M": {
        "colour": "Black",
        "size": "M",
        "currency": "USD",

        "price": 1995,
        "previous_prices": [5995]
    }
}
```

### Multiple Reductions ("Was $z, then $y, now $x")
Similar to a reduction as above, but where a product has been discounted by the retailer more than once. Simiarly, record a **list** of `previous_prices` for each SKU ensuring that prices are in chronological order (earliest price goes first).

```json
{
    "Black M": {
        "colour": "Black",
        "size": "M",
        "currency": "USD",

        "price": 1995,
        "previous_prices": [7995, 5995]
    }
}
```

### Percentage Reduction ("y% off, now $x")
In these cases, a price is displayed along with a percentage that this product was reduced by. In these cases, we simply want to record the percentage discount displayed.

Imagine that we have a product that has a price of $24.00 and is 20% off the original price.

```json
{
    "Blue S": {
        "colour": "Blue",
        "size": "S",
        "currency": "GBP",

        "price": 2400,
        "percentage_discount": 20
    }
}
```

(This means that the previous price must have been $30.00, but because the site
does not mention the value of $30, we only capture the values it __does__
mention: namely $24 and 20% off.)

## Notes on what to capture (1)

The aim of the crawling stage is to capture information. Processing is done
later on.

So as a general rule, __try to avoid performing calculations in the crawler__.

An example is: a product can have a previous price, a price and a discount
amount. Obviously, given two of the three values, the third can be calculated.

When a retailer only lists two of the three values, only capture those two.
I.e. do not worry about calculating the third.

Most of the time this will be when the price and discount amount is listed,
but sometimes a retailer will only list a previous price and discount amount.
In this case, do not calculate and add the price to the `skus`, simply capture
the known information.

## Notes on what to capture (2)

Sometimes a retailer will have a discount but it is not available in the HTML.
It could only be displayed within an image(!) and the only way you would find
out that there is a discount is by adding the product to the basket and
checking the total price.

For now, we do not require crawlers to have to find that information. I.e. __if
the discount can only be found by adding a item to the basket then do not
attempt to capture the discount__.

## Examples

For example, [this page](dont_capture.png) only has the discount listed in the
product image. It is not expected for a crawler to capture this or attempt to
calculate this.

On the other hand, [this page](do_capture.png) does have the previous price and
the current price in the HTML. We should capture both those but not attempt to
add the discount (as later stages in the pipeline can do so).

Another example is [this page](dont_capture_52_percent.png). The only reliable
price listing is `previous_price: 34800, price: 9999`.
The text about 52% off is a limited offer and can't be verified as a price.
A good rule-of-thumb is that if someone were to glance at that page, what would
they see the price to be? They would most likely see `$99.99`.
Thus the text about 52% can be included perhaps in merchandising info, but not
as a merchandised price. (And the crawler shouldn't calculate 52% of the
price.)

# Pipelines

All the crawled Garment items go trough a series of pipelines. The order
is specified in **settings.py** and the pipleines are defined in **pipelines.py**

```python
# from settings.py
ITEM_PIPELINES = [
  'skuscraper.pipelines.RequiredKeys',
  'skuscraper.pipelines.FixEscapedChars',
  'skuscraper.pipelines.NormalisePricesPipeline',
  'skuscraper.pipelines.MarketAndCurrencyMatchUp',
  'skuscraper.pipelines.ErrorCheckingPipeline',
  'skuscraper.pipelines.IsValidCurrency',
  'skuscraper.pipelines.MongoFySkuKeys',
  'skuscraper.pipelines.FixNoSpacesInImageUrls',
  'skuscraper.pipelines.ProductDispatcherPipeline',
  'skuscraper.pipelines.NoNewlineInImportantFields',
]
```

If any step in the pipelines fails the Garment will be thrown away.

**RequiredKeys** makes sure that all the required fields are in a scraped Garment.
For a complete out of stock item the requirements are different from a non out of stock item. This pipeline uses required_fields.py

**FixEscapedChars** fixes escaped chars in the name, brand, description and care fields.

```
&amp; -> &
&Amp; -> &
&pos; -> '
&quot; -> "
```

**NormalisePricesPipeline** takes the lowest price for a sku at sets the
garment['price'] with this value. It also sets the
garment['currency'] from the skus.

**MarketAndCurrencyMatchUp** makes sure that a spider (for example)
thas has market set as UK picks up currency as GBP, otherwise the
Garment is dropped

**ErrorCheckingPipeline** drops Garments that are in stock but are
missing currency and Garments where one or more of the skus are
missing currency.

**IsValidCurrency** checks that the currency we picked up is a valid one.
Currency "codes" are gathered from http://en.wikipedia.org/wiki/ISO_4217#Active_codes

**MongoFySkuKeys** makes sure that there is not any unsupported keys in
the Garment keys.

```
The '$' character must not be the first character in the key name.
The '.' character must not appear anywhere in the key name.
```

**FixNoSpacesInImageUrls** replaces space(' ') with '%20' in the image
URLs.

**“DispatchProductToSummariser** puts the crawled Garment on the
settings.SUMMARISER_QUEUE, this will enable the syncd workers to sync it

**NoNewlineInImportantFields** removes newlines and strip()'s in the name
and brand field.

NoNewlineInImportantFields should be called
before ProductDispatchPipeline!








# An example site and spider

There is an example site that has been set up purely to show how a spider works.
Its URL is http://retailer.edtd.net. Note that it is not a real retailer site!

The relevant spider for it is in skuscraper, called [`editd_spider`](spiders/editd_spider.py)

[https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py)

We can run a trial crawl from a skuscraper vagrant machine.

```
(venv)vagrant@skuscraper:/opt/skuscraper$ scrapy crawl EDITD-crawl &> crawl.log
```

As we can see from the end of the log file, the spider takes about a minute to run (the site only has 1200 products).

```
(venv)vagrant@skuscraper:/opt/skuscraper$ tail -n 18 crawl.log
2014-08-28 09:19:14+0000 [EDITD-crawl] INFO: Closing spider (finished)
2014-08-28 09:19:14+0000 [EDITD-crawl] INFO: Dumping spider stats:
	{'downloader/request_bytes': 383622,
	 'downloader/request_count': 1235,
	 'downloader/request_method_count/GET': 1235,
	 'downloader/response_bytes': 16292083,
	 'downloader/response_count': 1235,
	 'downloader/response_status_count/200': 1235,
	 'finish_reason': 'finished',
	 'finish_time': datetime.datetime(2014, 8, 28, 9, 19, 14, 561193),
	 'item_scraped_count': 1200,
	 'request_depth_max': 35,
	 'scheduler/memory_enqueued': 1235,
	 'start_time': datetime.datetime(2014, 8, 28, 9, 18, 38, 876531)}
2014-08-28 09:19:14+0000 [EDITD-crawl] INFO: sending stats to rabbitmq.
2014-08-28 09:19:14+0000 [EDITD-crawl] INFO: Spider closed (finished)
2014-08-28 09:19:14+0000 [scrapy] INFO: Dumping global stats:
	{'memusage/max': 829739008, 'memusage/startup': 829739008}
(venv)vagrant@skuscraper:/opt/skuscraper$
```

Let us look at the spider in detail.

## The Mixin

[https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L11-L15](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L11-L15)

```python
class Mixin(object):
    retailer = "EDITD"
    market = "UK"
    allowed_domains = ["retailer.edtd.net"]
    start_urls = ["http://retailer.edtd.net"]
```

Usually we have these in spiders as a way to ensure that the crawl spider and the parse spider both have the same `retailer` and `market`.
As you will see later, both extend `Mixin`.


## EDITDCrawlSpider

[https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L82](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L82)

This is the 'entry point'. The crawl spider gets its `start_urls` from the `Mixin`.

```python
class EDITDCrawlSpider(BaseCrawlSpider, Mixin):
    name = Mixin.retailer + "-crawl"
    parse_spider = EDITDParseSpider()

    listings_x = [
        "//a[contains(@class, 'listings')]",
        "//a[@class='next']",
    ]

    products_x = [
        "//div[contains(@class, 'item-row')]//a"
    ]

    rules = (
        Rule(SgmlLinkExtractor(restrict_xpaths=listings_x), callback='parse'),
        Rule(SgmlLinkExtractor(restrict_xpaths=products_x), callback='parse_item')
    )
```

The `listings_x` is a list of patterns. The spider will use these patterns to find links to listings pages.
E.g.  `"//a[contains(@class, 'listings')]"` is used to find the link on the home page [http://retailer.edtd.net/](http://retailer.edtd.net/) that leads us to the listings page.

Once we are on the first listing page, we need to keep clicking on any 'next page' links we see. In this case, those links are of the form `"//a[@class='next']"`.

So the `listings_x` list tells us where to find listings pages.

The `products_x` tells us where to find links on a listings page that lead to products.
So in this case, on the listing page, all the products are listed in the div that has the classname `item-row`.


## EDITDParseSpider

[https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L18](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L18)

Once the crawl spider finds product pages, products are extraced from those pages using the Parse Spider.

`parse` is called once for each product.

[https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L18-L35](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L18-L35)

```python
class EDITDParseSpider(BaseParseSpider, Mixin):
    name = Mixin.retailer + "-parse"

    def parse(self, response):
        hxs = HtmlXPathSelector(response)

        sku_id = self.product_id(hxs)

        garment = Garment()
        if self.duplicate(garment, sku_id):
            return

        self.boilerplate_normal(garment, hxs, response)

        garment["image_urls"] = self.image_urls(hxs)
        garment["skus"] = self.skus(hxs)

        return garment
```

It calls the `boilerplate_normal` function (which it inherits from [`BaseParseSpider`](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/base.py#L268)).

This expects the spider to provide various functions to extract the relevant fields (`name`, `description`, `product_id` etc), which the spider does.

[https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L37-L53](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L37-L53)

Finally, it does the extra things that are required, but not supplied by `boilerplate_normal`, namely the `image_urls` and the `skus`.
Don't forget those two fields! They're essential.

[https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L32-L33](https://github.com/EDITD/skuscraper/blob/8d9126d2e483c3e4cb1d972b97171ea572fccf87/spiders/editd_spider.py#L32-L33)



# Common pitfalls

Over the years, we have encountered several issues. Here are some of the important ones to avoid, and some advice that we have picked up on how to deal with them.


## Maintain product_hash

The retailer usually gives each of their products a unique identifier (e.g. 4125893).

We combine that with the `retailer` field of the crawler to get our own unique identifier.

(This way, if ASOS have a product that they've given the id of 4125, but debenhams also have a product with an id of 4125, they won't clash in our system, as we will assign them `asos_4125` and `debenhams_4125` respectively)

In the crawler, we hash this unique identifier as the `product_hash` field.

[https://github.com/EDITD/skuscraper/blob/master/spiders/base.py#L55-L56](https://github.com/EDITD/skuscraper/blob/master/spiders/base.py#L55-L56)

And later on (outside the `skuscraper` code), we produce another id based upon the same `retailer` and `reatiler_id` id.

**So it is important that we do not allow this value to change.**

So where possible, always try to maintain this.

What this means is:

* If we want to change the `retailer` key of a crawler after it has been run at least once then we need to be careful to preserve the old product hashes
* Change the `retailer` to the new name and make sure that Util script has been run before starting any new crawl with the new name; Util script converts all the products in the app to new retailer name giving new product hashes/id's `
* Be very careful about what to choose for a product identifier within a retailer. Once we pick it, it's very difficult to change it.
* If in any doubt, please ask!


## Only pick up details for the main product

Sometimes a retailer will include 'Suggested products' on the sidebar of a page.

It is very important that we **do not** accidentally pick up the price or name etc of the suggested product!

E.g. look at the this screenshot.

[https://s3-eu-west-1.amazonaws.com/suj-foo/many_prices.png](https://s3-eu-west-1.amazonaws.com/suj-foo/many_prices.png)

The correct price of the shorts is £12, but a careless selector might pick up the price of the product in the sidebar (not the main one).

Often prices, etc are defined using a selector like this:

```
//span[@class="NowPrice"])/text()
```

Be careful! This might match the price for the suggested product as well!

Where possible, always try to ensure that it is the main product that we select. Usually the sites will have a div called 'main' or 'mainDiv' or something similar. So the following is much safer!

```
//div[@class="mainDiv"]/span[@class="NowPrice"]/text()
```


## Report image urls in order

Often a retailer will have several images for a product.

However, they will usually have a 'main' image for the product. They always list this first (or have that image preselected in a carousel).

This is the image that we analyse for colours etc.

So it is important that whenever we extract image urls, we extract them in the order they are listed on the website.

So be very careful about using sets or dicts (or anything that would result in the order being lost).

## No data should be repeated between data points.

When we manipulate data to assign information to certain data points we don't want to repeat the information.

Common occurrences:
- When detecting brand from name, remove brand from name field.
- When detecting merch info from the description, remove merch info from description. 

## Data in general

In general, be very careful about crawling bad data! Incorrect data is something that can affect us greatly.

Always check your changes by running a crawl locally.


# HTTP and Networking


## User Agent Headers

While the standard scrapy settings for `USER_AGENT` still apply, there is a list of
common user agent strings: `settings.USER_AGENT_LIST`.

A spider can use randomise the user agent strings. To do so, simply add an attribute to the
spider: `randomise_user_agent`.

This should be

- `settings.RANDOMISE_USER_AGENT_PER_REQUEST`: Each request chooses a random user agent from the list.
  __This is the default.__
- `settings.RANDOMISE_USER_AGENT_ON_STARTUP`: When the crawler starts, choose a random user agent
  from the list and use that throughout the crawl
- `settings.RANDOMISE_USER_AGENT_NEVER`: Never randomise and just use the standard scrapy user
  agent values.

Note: if the spider has a specific `user_agent` attribute, then we do not randomise. (That is used instead.)
