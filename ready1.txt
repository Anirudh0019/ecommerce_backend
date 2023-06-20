create table customer
(user_id number primary key,
name varchar(50) not null,
mobile number(10) not null,
hostel varchar(25) not null)

create table LOGIN
(username varchar(50) primary key,
password varchar(50) not null)

CREATE TABLE USER_LOGIN (
  user_id NUMBER,
  username VARCHAR(50),
  CONSTRAINT fk_customer_user_id FOREIGN KEY (user_id) REFERENCES customer(user_id) ON DELETE CASCADE,
  CONSTRAINT fk_login_username FOREIGN KEY (username) REFERENCES LOGIN(username) ON DELETE CASCADE,
  PRIMARY KEY (user_id, username)
);

alter table ADVERTISEMENT add  user_id number ;

DESC ADVERTISEMENT

ALTER TABLE ADVERTISEMENT
ADD CONSTRAINT fk_advertisement_customer
FOREIGN KEY (user_id)
REFERENCES CUSTOMER(user_id);

    
create table ADVERTISEMENT 
(add_id number primary key,
title varchar(50) not null,
description varchar(500),
price number(10,2) not null,
age number(2),
category varchar(50) not null,
date_published date not null,
    check (category in('lab_coat','bicyle','drafter')) );

CREATE TABLE BOOKMARK (
  ADD_ID NUMBER,
  USER_ID NUMBER,
  CONSTRAINT FK_ADVERTISEMENT FOREIGN KEY (ADD_ID) REFERENCES ADVERTISEMENT(ADD_ID) ON DELETE CASCADE,
  CONSTRAINT FK_CUSTOMER FOREIGN KEY (USER_ID) REFERENCES CUSTOMER(USER_ID) ON DELETE CASCADE
);


create table REVIEW
(add_id number ,
rating number(2,1),
comments varchar(500),
CONSTRAINT FK_ADVERTISEMENT_review FOREIGN KEY (ADD_ID) REFERENCES ADVERTISEMENT(ADD_ID) ON DELETE CASCADE
    )

create table REQUIREMENT
(user_id number primary key,
category varchar(50),
CONSTRAINT FK_CUSTOMER_requirement FOREIGN KEY (user_id) REFERENCES CUSTOMER(user_id) ON DELETE CASCADE
    )



--hello there these are the procedures deployed 
-- procedure to insert into table
 CREATE OR REPLACE NONEDITIONABLE PROCEDURE "SYS"."INSERT_USER" (
    p_col2
IN customer.name%TYPE,
    p_col3 IN customer.mobile%TYPE,
    p_col4 IN
customer.hostel%TYPE,
        p_col5 IN LOGIN.username%TYPE,
    p_col6 IN
LOGIN.password%TYPE
)
IS
    l_user_id customer.user_id%TYPE;
BEGIN
    SELECT
user_id_seq.NEXTVAL INTO l_user_id FROM DUAL;
    INSERT INTO customer (user_id,
name, mobile, hostel)
    VALUES (l_user_id, p_col2, p_col3, p_col4);
    INSERT
INTO LOGIN(username,password )
    VALUES(p_col5,p_col6);
    INSERT INTO
USER_LOGIN(user_id,username)
    VALUES(l_user_id,p_col5);
    COMMIT;

DBMS_OUTPUT.PUT_LINE('User ID: ' || l_user_id);
END;
--/////////////////////////end 1
-- procedure to 








--function to return the total number of ads under a user_id 
CREATE OR REPLACE FUNCTION getadcount (
    p_user_id IN USER_LOGIN.user_id%TYPE
)
RETURN NUMBER
IS
    l_user_id USER_LOGIN.user_id%TYPE;
    l_username USER_LOGIN.username%TYPE;
    l_ad_count NUMBER;
BEGIN
    -- Retrieve the user_id and username of the specified user_id
    SELECT user_id, username INTO l_user_id, l_username
    FROM USER_LOGIN
    WHERE user_id = p_user_id;

    -- Retrieve the count of ads for the specified user_id
    SELECT COUNT(*) INTO l_ad_count
    FROM ADVERTISEMENT
    WHERE user_id = l_user_id;

    -- Display the user's username, user_id, and ad count
    DBMS_OUTPUT.PUT_LINE('Username: ' || l_username);
    DBMS_OUTPUT.PUT_LINE('User ID: ' || l_user_id);
    DBMS_OUTPUT.PUT_LINE('Ad Count: ' || l_ad_count);

    -- Return the ad count
    RETURN l_ad_count;
END;
/
DECLARE
    ad_count NUMBER;
BEGIN
    ad_count := getadcount(login_package.user_id);
    DBMS_OUTPUT.PUT_LINE('Ad Count: ' || ad_count);
END;

--- 
    CREATE SEQUENCE p_add_id_seq
    START WITH 1
    INCREMENT BY 1
    NOCACHE
    NOCYCLE;


----
CREATE OR REPLACE PROCEDURE add_advertisement(
    p_title IN advertisement.title%TYPE,
    p_description IN advertisement.description%TYPE,
    p_price IN advertisement.price%TYPE,
    p_age IN advertisement.age%TYPE,
    p_category IN advertisement.category%TYPE,
    p_date_published IN advertisement.date_published%TYPE,
    p_user_id IN advertisement.user_id%TYPE
)
AS
    l_add_id advertisement.add_id%TYPE;
BEGIN
    -- Get the next value from the sequence for add_id
    SELECT p_add_id_seq.NEXTVAL INTO l_add_id FROM DUAL;

    -- Insert the new advertisement into the table
    INSERT INTO advertisement (add_id, title, description, price, age, category, date_published, user_id)
    VALUES (l_add_id, p_title, p_description, p_price, p_age, p_category, p_date_published, p_user_id);

    -- Output a success message
    DBMS_OUTPUT.PUT_LINE('Advertisement added successfully.');
EXCEPTION
    -- Output an error message if an exception is raised
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('An error occurred: ' || SQLERRM);
END;
/
----
CREATE OR REPLACE PACKAGE login_package IS
    user_id NUMBER;
END login_package;
/

CREATE OR REPLACE PROCEDURE login_check(
    p_username IN VARCHAR2,
    p_password IN VARCHAR2
) AS
    v_password_from_db VARCHAR2(50);
    p_user_id NUMBER;
BEGIN
    SELECT password INTO v_password_from_db FROM login WHERE username = p_username;

    IF v_password_from_db = p_password THEN
        SELECT user_id INTO login_package.user_id FROM user_login WHERE username=p_username;
        DBMS_OUTPUT.PUT_LINE('Welcome, ' || p_username);
    ELSE
        RAISE_APPLICATION_ERROR(-20001, 'Incorrect username or password');
    END IF;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RAISE_APPLICATION_ERROR(-20002, 'Username ' || p_username || ' not found');
    WHEN OTHERS THEN
        RAISE_APPLICATION_ERROR(-20003, 'An error occurred: ' || SQLERRM);
END;


-----
CREATE OR REPLACE PROCEDURE add_bookmark
(add_id IN NUMBER,
 user_id IN NUMBER) 
AS
BEGIN
  INSERT INTO bookmark (add_id, user_id) VALUES (add_id, user_id);
END;

exec add_bookmark(1,21);

select* from advertisement;


CREATE OR REPLACE PROCEDURE set_requirement
(user_id IN NUMBER,
 category IN ADVERTISEMENT.category%TYPE)
AS
BEGIN
  INSERT INTO requirement (user_id, category) VALUES (user_id, category);
END;
/

CREATE OR REPLACE PROCEDURE give_review
(add_id IN NUMBER,
 rating IN REVIEW.rating%TYPE,
 comments IN REVIEW.comments%TYPE)
AS
BEGIN
  INSERT INTO review (add_id, rating, comments) VALUES (add_id, rating, comments);
END;


---

CREATE OR REPLACE PACKAGE add_package IS
    add_id NUMBER;
END add_package;
/
    
---
CREATE OR REPLACE PROCEDURE kholo_add(
    ad_id IN NUMBER
) AS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Ad ID: ' || ad_id);
	add_package.add_id:=ad_id;
    FOR review_row IN (SELECT rating, comments FROM review WHERE add_id = ad_id)
    LOOP
        DBMS_OUTPUT.PUT_LINE('Rating: ' || review_row.rating);
        DBMS_OUTPUT.PUT_LINE('Comments: ' || review_row.comments);
    END LOOP;
END;

/

----
----
--- 27 - 04 - 2023

CREATE OR REPLACE TRIGGER remove_ad_trigger
AFTER DELETE ON myuser.advertisements
FOR EACH ROW
BEGIN
  DELETE FROM myuser.bookmark
  WHERE ad_id = :old.add_id;
END;
/


-------------------------
    -
DECLARE
    ad_count NUMBER;
BEGIN
    ad_count := getadcount(21);
    DBMS_OUTPUT.PUT_LINE('Ad Count: ' || ad_count);
END;
/
---------------------------
DECLARE
  result NUMBER;
BEGIN
  result := get_bookmark_count(login_package.user_id); -- Replace 123 with the user ID you want to check
  DBMS_OUTPUT.PUT_LINE('Number of bookmarks added: ' || result);
END;
/
--------------------------

CREATE OR REPLACE PROCEDURE display_ads_by_category(p_category IN VARCHAR2) AS
    CURSOR ads_cursor IS
        SELECT add_id, title, description, price, age, date_published
        FROM advertisement
        WHERE category = p_category;
    v_add_id advertisement.add_id%TYPE;
    v_title advertisement.title%TYPE;
    v_description advertisement.description%TYPE;
    v_price advertisement.price%TYPE;
    v_age advertisement.age%TYPE;
    v_date_published advertisement.date_published%TYPE;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Ads under category ' || p_category || ':');
    DBMS_OUTPUT.PUT_LINE('-----------------------------');
    
    -- Loop through the cursor and display each ad
    FOR ad IN ads_cursor LOOP
        v_add_id := ad.add_id;
        v_title := ad.title;
        v_description := ad.description;
        v_price := ad.price;
        v_age := ad.age;
        v_date_published := ad.date_published;
        
        DBMS_OUTPUT.PUT_LINE('Ad ID: ' || v_add_id);
        DBMS_OUTPUT.PUT_LINE('Title: ' || v_title);
        DBMS_OUTPUT.PUT_LINE('Description: ' || v_description);
        DBMS_OUTPUT.PUT_LINE('Price: ' || v_price);
        DBMS_OUTPUT.PUT_LINE('Age: ' || v_age);
        DBMS_OUTPUT.PUT_LINE('Date published: ' || v_date_published);
        DBMS_OUTPUT.PUT_LINE('-----------------------------');
    END LOOP;
    
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('No ads found under category ' || p_category);
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('An error occurred: ' || SQLERRM);
END;


