
# Online Shop Database Management Project
As a part of our University Suleyman Demirel, we made this project for Database Management Systems 2 (DBMS)<br>

## Pre-requisite

Oracle SQL Server (or) Oracle Community Edition

## Contents

- Mini world and Project Description
- Basic structure
  - Functional requirements
  - Entity Relation (ER) diagram and constraints
  - Relational database schema
- Implementation
  - Creating tables
  - Inserting data
- Queries
  - Basic queries
  - PL/SQL function
  - Trigger function
  
 ### 2.2 Entity Relation Diagram
 ![Relational diagram](https://github.com/thevarp19/midterm_database_onlineShop/raw/main/ERD.jpg)
 
 ## 3. Implementation

You can directly copy and paste all the commands from the text given here into the SQL console to create and insert values into your table.


### 3.1 Creating Tables
 
 CREATE TABLE Payment (
  payment_id NUMBER PRIMARY KEY,
  payment_method VARCHAR2(50),
  payment_date DATE,
  payment_status VARCHAR2(50)
);

CREATE TABLE Courier (
  courier_id NUMBER PRIMARY KEY,
  courier_name VARCHAR2(50),
  courier_phone VARCHAR2(20),
  courier_email VARCHAR2(100)
);

CREATE TABLE Buyer (
  buyer_id NUMBER PRIMARY KEY,
  buyer_name VARCHAR2(50),
  buyer_email VARCHAR2(100),
  buyer_phone VARCHAR2(20),
  buyer_address VARCHAR2(200)
);

CREATE TABLE Orders (
  order_id NUMBER PRIMARY KEY,
  order_date DATE,
  order_status VARCHAR2(50),
  buyer_id NUMBER,
  payment_id NUMBER,
  courier_id NUMBER,
  FOREIGN KEY (buyer_id) REFERENCES Buyer(buyer_id),
  FOREIGN KEY (payment_id) REFERENCES Payment(payment_id),
  FOREIGN KEY (courier_id) REFERENCES Courier(courier_id)
);

CREATE TABLE OrderDetail (
  order_id NUMBER,
  product_id NUMBER,
  quantity NUMBER,
  price NUMBER,
  FOREIGN KEY (order_id) REFERENCES Orders(order_id),
  FOREIGN KEY (product_id) REFERENCES Product(product_id)
);

CREATE TABLE Product (
  product_id NUMBER PRIMARY KEY,
  product_name VARCHAR2(50),
  product_brand VARCHAR2(50),
  product_price NUMBER,
  product_description VARCHAR2(200),
  product_image VARCHAR2(100)
);

CREATE TABLE "Online Store" (
  store_id NUMBER PRIMARY KEY,
  store_name VARCHAR2(50),
  store_email VARCHAR2(100),
  store_phone VARCHAR2(20),
  store_address VARCHAR2(200)
);

CREATE TABLE Reports (
  report_id NUMBER PRIMARY KEY,
  report_type VARCHAR2(50),
  report_date DATE,
  report_description VARCHAR2(200)
);


---Foreign keys
ALTER TABLE orders
ADD CONSTRAINT fk_buyer
  FOREIGN KEY (buyer_id) REFERENCES buyer(buyer_id)
  ON DELETE CASCADE;

ALTER TABLE orders
ADD CONSTRAINT fk_payment
  FOREIGN KEY (payment_id) REFERENCES payment(payment_id)
  ON DELETE CASCADE;

ALTER TABLE orders
ADD CONSTRAINT fk_courier
  FOREIGN KEY (courier_id) REFERENCES courier(courier_id)
  ON DELETE CASCADE;

  ALTER TABLE orders
ADD CONSTRAINT fk_product
  FOREIGN KEY (product_id) REFERENCES product(product_id)
  ON DELETE CASCADE;

  ALTER TABLE reports
ADD CONSTRAINT fk_report
  FOREIGN KEY (order_id) REFERENCES orders(order_id)
  ON DELETE CASCADE;

  ALTER TABLE return
ADD CONSTRAINT fk_return
  FOREIGN KEY (order_id) REFERENCES order_id(order_id)
  ON DELETE CASCADE;

  ALTER TABLE feedback
ADD CONSTRAINT fk_buyer
  FOREIGN KEY (buyer_id) REFERENCES product(buyer_id)
  ON DELETE CASCADE;
  
  ALTER TABLE feedback
ADD CONSTRAINT fk_product
  FOREIGN KEY (product_id) REFERENCES product(product_id)
  ON DELETE CASCADE;



create or replace PACKAGE total_price AS    
    PROCEDURE print_revenue;
END total_price;


create or replace PROCEDURE cancel_order (order_id IN ORDERS.order_id%TYPE) 
IS
BEGIN
  DELETE FROM ORDERDETAIL WHERE order_id = cancel_order.order_id;
  
  DELETE FROM ORDERS WHERE order_id = cancel_order.order_id;
  
  DBMS_OUTPUT.PUT_LINE('Order ' || order_id || ' has been cancelled and its associated order details and payment records have been deleted.');
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('An error occurred: ' || SQLERRM);
END;

create or replace PROCEDURE print_product_info(p_product_name IN PRODUCT.product_name%TYPE)
IS      
    least_five_exception exception;
    v_product_id PRODUCT.product_id%TYPE;
    v_product_price PRODUCT.product_price%TYPE;
    v_product_description PRODUCT.product_description%TYPE;
    v_product_quantity PRODUCT.quantity%type;
BEGIN
    IF LENGTH(p_product_name) < 5 THEN
        RAISE least_five_exception;
    END IF;
    
    SELECT product_id, product_price, product_description
    INTO v_product_id, v_product_price, v_product_description
    FROM PRODUCT
    WHERE product_name = p_product_name;
    
    DBMS_OUTPUT.PUT_LINE('Product ID: ' || v_product_id);
    DBMS_OUTPUT.PUT_LINE('Product Name: ' || p_product_name);
    DBMS_OUTPUT.PUT_LINE('Product Price: ' || v_product_price);
    DBMS_OUTPUT.PUT_LINE('Product Description: ' || v_product_description);
EXCEPTION
    WHEN least_five_exception THEN
         DBMS_OUTPUT.PUT_LINE('Product name should be at least 5 characters.');
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Product not found.');
END;
/
create or replace PROCEDURE update_buyer_phone (
    p_buyer_id IN BUYER.buyer_id%TYPE,
    p_phone IN BUYER.buyer_phone%TYPE
)
IS
BEGIN
    UPDATE BUYER
    SET buyer_phone = p_phone
    WHERE buyer_id = p_buyer_id;

    IF SQL%ROWCOUNT = 1 THEN
        DBMS_OUTPUT.PUT_LINE('Buyer phone number updated successfully.');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Buyer not found or phone number already up to date.');
    END IF;
END;
/
create or replace FUNCTION get_total_orders_for_buyer(buyer IN NUMBER)
RETURN NUMBER
IS
  total_orders NUMBER;
BEGIN
  SELECT COUNT(*) INTO total_orders FROM ORDERS WHERE BUYER_ID = buyer;
  RETURN total_orders;
  IF SQL%ROWCOUNT=0 then
    raise NO_DATA_FOUND;
    END IF;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    DBMS_OUTPUT.PUT_LINE('No orders found for buyer with ID ' || buyer);
    RETURN 0;
END;
/
create or replace FUNCTION totalBuyers 
RETURN number IS 
   total number := 0; 
BEGIN 
   SELECT count(*) into total 
   FROM buyer; 
    
   RETURN total; 
END;
/
create or replace TRIGGER buyer_count_trigger
BEFORE INSERT ON buyer
DECLARE
  buyer_count NUMBER;
BEGIN
  SELECT COUNT(*) INTO buyer_count FROM buyer;
  DBMS_OUTPUT.PUT_LINE('The current number of rows in the BUYERS table is: ' || buyer_count);
END;
/
create or replace TRIGGER update_orderdetail 
BEFORE INSERT ON ORDERDETAIL 
FOR EACH ROW 
DECLARE 
v_price_order NUMBER;
v_cost NUMBER;

BEGIN
SELECT P.PRODUCT_PRICE INTO v_price_order FROM PRODUCT P
JOIN ORDERS ORD ON P.PRODUCT_ID = ORD.PRODUCT_ID 
WHERE ORD.PRODUCT_ID = :NEW.PRODUCT_ID AND ORD.ORDER_STATUS = 'Complete';
v_cost := v_price_order * :NEW.quantity;
:NEW.price := v_cost;
END;
/
create or replace TRIGGER update_product_quantity 
BEFORE INSERT ON ORDERS
FOR EACH ROW 
DECLARE 
  v_quantity NUMBER;
  v_stock_quantity NUMBER;
BEGIN
  SELECT quantity INTO v_quantity FROM PRODUCT
  WHERE PRODUCT_ID = :NEW.PRODUCT_ID;

  IF :NEW.ORDER_STATUS = 'Complete' THEN
    v_stock_quantity := v_quantity - :NEW.quantity;
    UPDATE PRODUCT
    SET quantity = v_stock_quantity
    WHERE PRODUCT_ID = :NEW.PRODUCT_ID;
  END IF;
END;
