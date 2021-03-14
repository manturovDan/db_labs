#Manturov's Trading House

###Database creation
```bigquery
CREATE DATABASE trading_house;
```

###Schema creation
```bigquery
CREATE SCHEMA trading;
```

###Tables creation
```bigquery
CREATE TABLE goods_group (
    group_id integer PRIMARY KEY NOT NULL,
    parent_group_id integer REFERENCES goods_group(group_id),
    name varchar(256) NOT NULL,
    description text
);

CREATE TABLE currency (
    currency_id integer PRIMARY KEY NOT NULL,
    name varchar(256) NOT NULL,
    code integer NOT NULL UNIQUE,
    country varchar(50),
    description text
);

CREATE TABLE currency_rate (
    rate_id integer PRIMARY KEY NOT NULL,
    date date NOT NULL,
    value double precision NOT NULL,
    numerator integer REFERENCES currency(currency_id),
    denominator integer REFERENCES currency(currency_id)
);

CREATE TABLE good (
    good_id integer NOT NULL,
    group_id integer REFERENCES goods_group(group_id) NOT NULL,
    PRIMARY KEY (good_id, group_id),
    name varchar(256) NOT NULL,
    vendor_code varchar(256) NOT NULL,
    count integer CHECK ( count >= 0 ),
    price money NOT NULL,
    certificate varchar(256),
    wrap varchar(256),
    vendor varchar(256) NOT NULL,
    currency integer NOT NULL REFERENCES currency(currency_id)
);

CREATE TABLE company_category (
    category_id integer PRIMARY KEY NOT NULL,
    name varchar(256) NOT NULL
);

CREATE TABLE client_company (
    company_id integer PRIMARY KEY NOT NULL,
    name varchar(256) NOT NULL,
    legal_address text NOT NULL ,
    phone varchar(50) NOT NULL,
    licence varchar(50) NOT NULL,
    requisites text NOT NULL,
    category integer REFERENCES company_category(category_id)
);

CREATE TYPE order_type AS ENUM ('release', 'return');

CREATE TABLE payment_document (
    document_id integer NOT NULL,
    department_id integer NOT NULL,
    type order_type NOT NULL,
    PRIMARY KEY (document_id, department_id, type),
    bank_id integer NOT NULL,
    datetime timestamp NOT NULL,
    amount money NOT NULL,
    currency integer REFERENCES currency(currency_id)
);

CREATE TABLE goods_document (
    document_id integer NOT NULL,
    type order_type NOT NULL,
    PRIMARY KEY (document_id, type),
    company integer REFERENCES client_company NOT NULL,
    datetime timestamp NOT NULL,
    base_document_id integer,
    base_document_type order_type CHECK ( base_document_type IS NULL OR base_document_type IN ('release')),
    FOREIGN KEY (base_document_id, base_document_type) REFERENCES goods_document(document_id, type), --check if base type is release
    payment_document_id integer NOT NULL,
    payment_department_id integer NOT NULL,
    payment_type order_type NOT NULL,
    FOREIGN KEY (payment_document_id, payment_department_id, payment_type)
            REFERENCES payment_document (document_id, department_id, type)
);

CREATE TABLE good_to_document (
    good_id integer NOT NULL,
    group_id integer NOT NULL,
    good_document_id integer NOT NULL,
    good_document_type order_type NOT NULL,
    FOREIGN KEY (good_id, group_id) REFERENCES good (good_id, group_id),
    FOREIGN KEY (good_document_id, good_document_type) REFERENCES goods_document (document_id, type),
    PRIMARY KEY (good_id, group_id, good_document_id, good_document_type),
    price_per_one money NOT NULL,
    count integer CHECK ( count > 0 ),
    currency integer NOT NULL REFERENCES currency(currency_id)
);
```