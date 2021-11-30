# TD 1 : Vendre des livres numériques avec WooCommerce

## 1. S'assurer d'avoir docker compose

```shell
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
mkdir -p ~/bin
cd ~/bin
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o ~/bin/docker-compose
# macOS avec processeurs ARM M1
# curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-x86_64" -o ~/bin/docker-compose
chmod +x ~/bin/docker-compose
source ~/.bashrc
# vérifier qu'on a docker-compose installé et qui tourne bien
docker-compose --version
# Docker Compose version 1.29.2
```

## 2. Démarrer un wordpress via docker compose

### Préparer l'environnement docker

```shell
# variables d'environnement pour contourner des limitations docker
grep -c 'export USER_ID' ~/.bashrc &>/dev/null || echo "export USER_ID=$(id -u)" >> ~/.bashrc
grep -c 'export GROUP_ID' ~/.bashrc &>/dev/null || echo "export GROUP_ID=$(id -g)" >> ~/.bashrc
source ~/.bashrc
```

```shell
# répertoires pour persister sur l'hôte local
mkdir -p td1
chmod a+rwx ~/
chmod a+rwx td1
cd td1
mkdir -p wordpress/src mysql/database
chmod a+rwx -R wordpress mysql
```

Créer le fichier `wordpress/Dockerfile`:
```dockerfile
FROM wordpress:latest
ARG USER_ID
ARG GROUP_ID
RUN sed -i -e "s/33:33/${USER_ID}:${GROUP_ID}/g" /etc/passwd \
  && sed -i -e "s/33:/${GROUP_ID}:/g" /etc/group \
  && chown -R www-data:www-data /var/www
```

Créer le fichier `mysql/Dockerfile`:
```dockerfile
FROM mysql:5.7
ARG USER_ID
ARG GROUP_ID
RUN sed -i -e "s/999:999/${USER_ID}:${GROUP_ID}/g" /etc/passwd \
  && sed -i -e "s/999:/${GROUP_ID}:/g" /etc/group \
  && chown -R mysql:mysql /var/lib/mysql
```

### Augmenter la taille des fichiers qu'on peut uploader dans wordpress


Créer le fichier `wordpress/aixmazone.ini`:
```ini
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 300
max_input_time=300
```

### Créer un fichier docker compose

```yaml
version: '3.8'

services:
    db:
        image: aixmazone/mysql
        build:
          context: mysql
          args:
            USER_ID: ${USER_ID}
            GROUP_ID: ${GROUP_ID}
        volumes:
            - ./mysql/database:/var/lib/mysql
        restart: always
        #user: mysql # uncomment on subsequent starts
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: aixmazone/wordpress
        build:
          context: wordpress
          args:
            USER_ID: ${USER_ID}
            GROUP_ID: ${GROUP_ID}
        depends_on:
            - db
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: db:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - ./wordpress/aixmazone.ini:/usr/local/etc/php/conf.d/aixmazone.ini:ro
            - ./wordpress/src:/var/www/html
```

### Copier les fichiers source de docker sur le filesystem local


```shell
docker-compose build
CID=$(docker create --entrypoint=scratch aixmazone/wordpress)
docker cp $CID:/usr/src/wordpress/. wordpress/src/
docker rm $CID
```

### Démarrer les services avec docker compose

```shell
docker-compose up
```

## 3. Installer l'extension Woocommerce

http://localhost:8080/wp-admin
Extension ⇒ Ajouter ⇒ chercher “woocommerce” ⇒ Installer.


# TD 2 : Créer un frontstore en typescript/nextjs pour son WooCommerce

## Initialiser un projet nextjs en typescript

```shell
npx create-next-app nextjs-frontstore --ts
cd nextjs-frontstore
```

## Installer la rest api woocommerce

```shell
npm install --save @woocommerce/woocommerce-rest-api
npm install --save @types/woocommerce__woocommerce-rest-api
mkdir -p infrastructure
```

### Créer le fichier `infrastructure/wooCommerceApi.ts`

```ts
import WooCommerceRestApi from "@woocommerce/woocommerce-rest-api";

// initialize the WooCommerceRestApi //
const api = new WooCommerceRestApi({
  url: "http://localhost:8080",
  consumerKey: 'cs_......', // ← mettez votre valeur générée
  consumerSecret: 'ck_........', // ← mettez votre valeur générée
  version: "wc/v3",
});

// fetch all products from WooCommerce //
export async function fetchWooCommerceProducts() {
  try {
    const response = await api.get("products");
    return response;
  } catch (error) {
    throw new Error(error);
  }
}
```

### Créer le fichier `domain/wooCommerceTypes.ts`

```ts
export interface Product {
  id: number;
  name: string;
  slug: string;
  permalink: string;
  date_created: Date;
  date_created_gmt: Date;
  date_modified: Date;
  date_modified_gmt: Date;
  type: "simple" | "variable" | "grouped" | "external";
  status: "any" | "draft" | "pending" | "private" | "publish";
  featured: boolean;
  catalog_visibility: "visible" | "catalog" | "search" | "hidden";
  description: string;
  short_description: string;
  sku: string;
  price: string;
  regular_price: string;
  sale_price: string;
  date_on_sale_from: Date | null;
  date_on_sale_from_gmt: Date | null;
  date_on_sale_to: Date | null;
  date_on_sale_to_gmt: Date | null;
  price_html: string;
  on_sale: boolean;
  purchasable: boolean;
  total_sales: number;
  virtual: boolean;
  downloadable: boolean;
  downloads: any[]; // TODO look at Downloads properties
  download_limit: number;
  download_expiry: number;
  external_url: string;
  button_text: string;
  tax_status: "taxable" | "shipping" | "none";
  tax_class: "standard" | "reduced-rate" | "zero-rate";
  manage_stock: boolean;
  stock_quantity: number;
  stock_status: "instock" | "outofstock" | "onbackorder";
  backorders: "no" | "notify" | "yes";
  backorders_allowed: boolean;
  backordered: boolean;
  sold_individually: boolean;
  weight: string;
  dimensions: Dimensions;
  shipping_required: boolean;
  shipping_taxable: boolean;
  shipping_class: string;
  shipping_class_id: number;
  reviews_allowed: boolean;
  average_rating: string;
  rating_count: number;
  related_ids: number[];
  upsell_ids: number[];
  cross_sell_ids: number[];
  parent_id: number;
  purchase_note: string;
  categories: Partial<Category>[];
  tags: any[]; // TODO look at Tags properties
  images: Image[];
  attributes: Attribute[];
  default_attributes: any[]; // TODO look at default attributes properties
  variations: number[];
  grouped_products: number[];
  menu_order: number;
  meta_data: MetaDatum[];
  _links: Links;
}

export interface Category {
  id: number;
  name: string;
  slug: string;
  parent: number;
  description: string;
  display: string;
  image: Image;
  menu_order: number;
  count: number;
  _links: Links;
}

export interface Collection {
  href: string;
}

export interface Dimensions {
  length: string;
  width: string;
  height: string;
}

export interface MetaDatum {
  id: number;
  key: string;
  value: string;
}

interface LineItem {
  id: number;
  name: string;
  product_id: number;
  variation_id: number;
  quantity: number;
  tax_class: string;
  subtotal: string;
  subtotal_tax: string;
  total: string;
  total_tax: string;
  taxes: any[];
  meta_data: MetaData[];
  sku: string;
  price: number;
}

interface ShippingLine {
  id: number;
  method_title: string;
  method_id: string;
  instance_id: string;
  total: string;
  total_tax: string;
  taxes: any[];
  meta_data: any[];
}

interface Meta_Data_Line_Item {
  // built from my own object sending in, disregard if necessary!
  key: string;
  value: string;
}

interface Cart {
  // built from my own object sending in, disregard if necessary!
  payment_method: string;
  payment_method_title: string;
  billing: Billing;
  shipping: Shipping;
  line_items: Array<LineItem>;
  shipping_lines: Array<ShippingLine>;
  customer_id: number;
  meta_data: Array<Meta_Data_Line_Item>;
  set_paid: false;
}

// interface Attribute {
// 	id: number;
// 	name: string;
// 	position: number;
// 	visible: boolean;
// 	variation: boolean;
// 	options: string[];
// }

export interface Image {
  id: number;
  date_created: Date;
  date_created_gmt: Date;
  date_modified: Date;
  date_modified_gmt: Date;
  src: string;
  name: string;
  alt: string;
  position: number;
}

export interface Attribute {
  id: number;
  name: string;
  option: string;
}

export interface MetaData {
  id: number;
  key: string;
  value: string;
}

export interface Up {
  href: string;
}

export interface Customer {
  id: number;
  date_created: Date;
  date_created_gmt: Date;
  date_modified: Date;
  date_modified_gmt: Date;
  email: string;
  first_name: string;
  last_name: string;
  role: string;
  username: string;
  billing: Billing;
  shipping: Shipping;
  is_paying_customer: boolean;
  avatar_url: string;
  meta_data: MetaData[];
  _links: Links;
}

export interface Order {
  id: number;
  parent_id: number;
  number: string;
  order_key: string;
  created_via: string;
  version: string;
  status: string;
  currency: string;
  date_created: Date;
  date_created_gmt: Date;
  date_modified: Date;
  date_modified_gmt: Date;
  discount_total: string;
  discount_tax: string;
  shipping_total: string;
  shipping_tax: string;
  cart_tax: string;
  total: string;
  total_tax: string;
  prices_include_tax: boolean;
  customer_id: number;
  customer_ip_address: string;
  customer_user_agent: string;
  customer_note: string;
  billing: Billing;
  shipping: Shipping;
  payment_method: string;
  payment_method_title: string;
  transaction_id: string;
  date_paid?: any;
  date_paid_gmt?: any;
  date_completed?: any;
  date_completed_gmt?: any;
  cart_hash: string;
  meta_data: any[];
  line_items: LineItem[];
  tax_lines: any[];
  shipping_lines: ShippingLine[];
  fee_lines: any[];
  coupon_lines: any[];
  refunds: any[];
  _links: Links;
}

export interface Links {
  self: Self[];
  collection: Collection[];
  up: Up[];
}

export interface Billing {
  first_name: string;
  last_name: string;
  company: string;
  address_1: string;
  address_2: string;
  city: string;
  state: string;
  postcode: string;
  country: string;
  email: string;
  phone: string;
}

export interface Shipping {
  first_name: string;
  last_name: string;
  company: string;
  address_1: string;
  address_2: string;
  city: string;
  state: string;
  postcode: string;
  country: string;
  email: string;
}

export interface Variation {
  id: number;
  date_created: Date;
  date_created_gmt: Date;
  date_modified: Date;
  date_modified_gmt: Date;
  description: string;
  permalink: string;
  sku: string;
  price: string;
  regular_price: string;
  sale_price: string;
  date_on_sale_from?: any;
  date_on_sale_from_gmt?: any;
  date_on_sale_to?: any;
  date_on_sale_to_gmt?: any;
  on_sale: boolean;
  visible: boolean;
  purchasable: boolean;
  virtual: boolean;
  downloadable: boolean;
  downloads: any[];
  download_limit: number;
  download_expiry: number;
  tax_status: string;
  tax_class: string;
  manage_stock: boolean;
  stock_quantity?: any;
  in_stock: boolean;
  backorders: string;
  backorders_allowed: boolean;
  backordered: boolean;
  weight: string;
  dimensions: Dimensions;
  shipping_class: string;
  shipping_class_id: number;
  image: Image;
  attributes: Attribute[];
  menu_order: number;
  meta_data: MetaData[];
  _links: Links;
}

export interface Self {
  href: string;
}
```


### Changer`le fichier `pages/index.tsx`

```tsx
import type { NextPage } from "next";
import Head from "next/head";
import Image from "next/image";
import { GetStaticProps } from "next";
import styles from "../styles/Home.module.css";
import { fetchWooCommerceProducts } from "../infrastructure/wooCommerceApi";
import { Product } from "../domain/wooCommerceTypes";
// import styled from "styled-components";
// import { ProductCard } from "../ui/productCard";

interface Props {
  products: Product[];
}

const Home: NextPage = (props: Props) => {
  const { products } = props;

  return (
    <div className={styles.container}>
      <Head>
        <title>Create Next App</title>
        <meta name="description" content="Generated by create next app" />
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main className={styles.main}>
        <h1>Welcome to Next.js!</h1>
        {products !== undefined &&
          products.map((product) => {
            return <div>{product.name}</div>;
            // return <ProductCard product={product} key={product.id} />;
          })}
      </main>

      <footer className={styles.footer}>
        <a
          href="https://vercel.com?utm_source=create-next-app&utm_medium=default-template&utm_campaign=create-next-app"
          target="_blank"
          rel="noopener noreferrer"
        >
          Powered by{" "}
          <span className={styles.logo}>
            <Image src="/vercel.svg" alt="Vercel Logo" width={72} height={16} />
          </span>
        </a>
      </footer>
    </div>
  );
};

export const getStaticProps: GetStaticProps = async () => {
  const wooCommerceProducts = await fetchWooCommerceProducts().catch((error) =>
    console.error(error)
  );

  if (!wooCommerceProducts) {
    return {
      notFound: true,
    };
  }

  console.log(wooCommerceProducts.data);

  return {
    props: {
      products: wooCommerceProducts.data,
    },
    // revalidate: 60 // regenerate page with new data fetch after 60 seconds
  };
};

export default Home;
```
