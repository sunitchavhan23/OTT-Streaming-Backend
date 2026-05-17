-- User Table
create table users(user_id int primary key,
                    full_name varchar2(100) not null,
                    email varchar2(150) not null unique,
                    password varchar2(255) not null,
                    phone varchar2(15) unique,
                    created_at timestamp default  current_timestamp,
                    status varchar2(20) DEFAULT 'active');

-- Subscription Plans Table
CREATE TABLE subscription_plans (
    plan_id NUMBER PRIMARY KEY,
    plan_name VARCHAR2(50) NOT NULL UNIQUE,
    price NUMBER(8,2) NOT NULL,
    duration_days NUMBER NOT NULL,
    video_quality VARCHAR2(20) NOT NULL
);

-- User Subscription Table
CREATE TABLE user_subscriptions (
    subscription_id NUMBER PRIMARY KEY,
    user_id NUMBER NOT NULL,
    plan_id NUMBER NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status VARCHAR2(20) DEFAULT 'active',
    CONSTRAINT fk_us_user FOREIGN KEY (user_id) REFERENCES users(user_id),
    CONSTRAINT fk_us_plan FOREIGN KEY (plan_id) REFERENCES subscription_plans(plan_id)
);

-- Content Table
CREATE TABLE content (
    content_id NUMBER PRIMARY KEY,
    title VARCHAR2(200) NOT NULL,
    description CLOB,
    release_year NUMBER,
    content_type VARCHAR2(20) NOT NULL,
    age_rating VARCHAR2(10),
    language VARCHAR2(50),
    duration_minutes NUMBER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Genres Table
CREATE TABLE genres (
    genre_id NUMBER PRIMARY KEY,
    genre_name VARCHAR2(50) NOT NULL UNIQUE
);

-- Content Genre Table
CREATE TABLE content_genre (
    content_id NUMBER NOT NULL,
    genre_id NUMBER NOT NULL,
    CONSTRAINT pk_cg PRIMARY KEY (content_id, genre_id),
    CONSTRAINT fk_cg_content FOREIGN KEY (content_id) REFERENCES content(content_id),
    CONSTRAINT fk_cg_genre FOREIGN KEY (genre_id) REFERENCES genres(genre_id)
);

-- Watch History Table
CREATE TABLE watch_history (
    history_id NUMBER PRIMARY KEY,
    user_id NUMBER NOT NULL,
    content_id NUMBER NOT NULL,
    watch_time NUMBER,
    watched_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_wh_user FOREIGN KEY (user_id) REFERENCES users(user_id),
    CONSTRAINT fk_wh_content FOREIGN KEY (content_id) REFERENCES content(content_id)
);

-- Payments Table
CREATE TABLE payments (
    payment_id NUMBER PRIMARY KEY,
    user_id NUMBER NOT NULL,
    subscription_id NUMBER NOT NULL,
    amount NUMBER(8,2) NOT NULL,
    payment_method VARCHAR2(50) NOT NULL,
    payment_status VARCHAR2(20) DEFAULT 'success',
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_pay_user FOREIGN KEY (user_id) REFERENCES users(user_id),
    CONSTRAINT fk_pay_sub FOREIGN KEY (subscription_id) REFERENCES user_subscriptions(subscription_id)
);

-- Insert into USERS
INSERT INTO users VALUES (1, 'Rahul Sharma', 'rahul@example.com', 'hashed123', '9876543210', TO_TIMESTAMP('2026-05-16 10:00:00', 'YYYY-MM-DD HH24:MI:SS'), 'ACTIVE');
INSERT INTO users VALUES (2, 'Priya Singh', 'priya@example.com', 'hashed456', '8765432109', TO_TIMESTAMP('2026-05-16 10:05:00', 'YYYY-MM-DD HH24:MI:SS'), 'ACTIVE');

-- Insert into SUBSCRIPTION_PLANS
INSERT INTO subscription_plans VALUES (1, 'Basic Plan', 199.00, 30, '720p');
INSERT INTO subscription_plans VALUES (2, 'Premium Plan', 499.00, 30, '4K');

-- Insert into USER_SUBSCRIPTIONS
INSERT INTO user_subscriptions VALUES (101, 1, 1, TO_DATE('2026-05-01', 'YYYY-MM-DD'), TO_DATE('2026-05-31', 'YYYY-MM-DD'), 'ACTIVE');
INSERT INTO user_subscriptions VALUES (102, 2, 2, TO_DATE('2026-05-10', 'YYYY-MM-DD'), TO_DATE('2026-06-09', 'YYYY-MM-DD'), 'ACTIVE');

-- Insert into CONTENT
INSERT INTO content VALUES (201, 'The Great Adventure', 'An epic journey across the seas.', 2024, 'Movie', 'PG-13', 'English', 120, TO_TIMESTAMP('2024-01-10 14:00:00', 'YYYY-MM-DD HH24:MI:SS'));
INSERT INTO content VALUES (202, 'Tech Startup', 'A thrilling series about young coders.', 2025, 'Series', 'TV-MA', 'Hindi', 45, TO_TIMESTAMP('2025-06-15 09:30:00', 'YYYY-MM-DD HH24:MI:SS'));

-- Insert into GENRES
INSERT INTO genres  VALUES (1, 'Action');
INSERT INTO genres VALUES (2, 'Drama');

-- Insert into CONTENT_GENRE
INSERT INTO content_genre  VALUES (201, 1);
INSERT INTO content_genre  VALUES (202, 2);

-- Insert into WATCH_HISTORY
INSERT INTO watch_history VALUES (301, 1, 201, 60, TO_TIMESTAMP('2026-05-16 11:00:00', 'YYYY-MM-DD HH24:MI:SS'));
INSERT INTO watch_history VALUES (302, 2, 202, 45, TO_TIMESTAMP('2026-05-16 11:15:00', 'YYYY-MM-DD HH24:MI:SS'));

-- Insert into PAYMENTS
INSERT INTO payments VALUES (401, 1, 101, 199.00, 'UPI', 'SUCCESS', TO_TIMESTAMP('2026-05-01 10:00:00', 'YYYY-MM-DD HH24:MI:SS'));
INSERT INTO payments VALUES (402, 2, 102, 499.00, 'Credit Card', 'SUCCESS', TO_TIMESTAMP('2026-05-10 10:00:00', 'YYYY-MM-DD HH24:MI:SS'));

-- 3. User Management Module
-- Add new user (Stored Procedure – Users table)
create or replace procedure register_user(
    p_user_id in int,
    p_full_name in varchar2,
    p_email in varchar2,
    p_password in varchar2,
    p_phone in varchar2
)is
begin
    insert into users(user_id,full_name,email,password,phone,created_at,status) values
                     (p_user_id,p_full_name,p_email,p_password,p_phone,current_timestamp,'active');
end;


begin
    register_user(3,'Sunit Chavhan','sunit@gmail.com','hashed123','9359436789');
end;

-- Update user details (Users table)
update users set phone='9226557849' where user_id=2;

-- Delete user (Users table)
delete from user_subscriptions where user_id=3;
delete from watch_history where user_id=3;
delete from payments where user_id=3;
delete from users where user_id=3;

-- Fetch all users (Users table)
select * from users;

-- Check duplicate email (Unique Key validation – Users table)
-- Because we added a UNIQUE constraint during table creation, Oracle will automatically prevent duplicates

-- Count total users (Aggregate Function COUNT – Users table)
select count(user_id) as total_users from users;

-- 4. Content Management Module
-- Add new content (Stored Procedure – Content table)
create or replace procedure add_content(
    p_content_id in int,
    p_title in varchar2,
    p_desc in clob,
    p_year in int,
    p_type in varchar2,
    p_rating in varchar2,
    p_lang in varchar2,
    p_duration in int
)is
begin
    insert into content(content_id,title,description,release_year,content_type,age_rating,language,duration_minutes,created_at)
    values (
    p_content_id ,p_title,p_desc,p_year,p_type,p_rating,p_lang,p_duration,current_timestamp
);
end;

begin
    add_content(203, 'Mirzapur', 'A thrilling series', 2021, 'Series', 'TV-MA', 'Hindi', 45);
end;

-- Update content (Content table)
update content set title='Mirzapur S1' where content_id=203;

-- Delete content (Content table)
delete from content where content_id=203;

-- Fetch content sorted by rating (ORDER BY – Content table)
select * from content
order by age_rating asc;

-- Get high-rated content (Simple View – Content table)
create or replace view
high_rated_content as
select * from content where age_rating in ('TV-MA','R');

select * from high_rated_content;

-- Find content released after specific year (Subquery – Content table)
select * from content where release_year >
(
   select avg(release_year) from content
);

-- 5. Subscription Plan Module
-- Add subscription plan (Stored Procedure – Subscription_Plans table)
create or replace procedure add_subscription_plan(
    p_plan_id in int,p_plan_name in varchar2,p_price in int,p_duration in int,p_quality in varchar2
)is
begin
    insert into subscription_plans(plan_id,plan_name,price,duration_days,video_quality)
    values (p_plan_id,p_plan_name,p_price,p_duration,p_quality);

end;

begin
    add_subscription_plan(3,'Standard',399,45,'FHD');
end;

-- Update plan details (Subscription_Plans table)
update subscription_plans set price=349 where plan_id=3;

-- Count number of plans (COUNT – Subscription_Plans table)
select count(*) as total_plans from subscription_plans;

-- Ensure plan name is unique (Unique Key – Subscription_Plans table)
-- This requirement is fulfilled structurally via the UNIQUE constraint on the plan_name column applied during the DDL phase (plan_name VARCHAR2(50) NOT NULL UNIQUE).

-- 6. Subscription Management Module
-- Assign subscription to user (Foreign Key – Users + Subscriptions)
create or replace procedure assign_subscription(
    p_sub_id in int,p_user_id in int,p_plan_id in int, p_start_date in date,p_end_date in date
)is
begin
    insert into user_subscriptions(subscription_id,user_id,plan_id,start_date,end_date,status)
    values(p_sub_id,p_user_id,p_plan_id,p_start_date,p_end_date,'active');

end;

-- Cancel Subscription
create or replace procedure cancel_subscription(p_sub_id in int)
is
begin
    update user_subscriptions set status='Cancelled' where subscription_id=p_sub_id;
end;


-- Fetch subscription details of user (JOIN – Users + Subscriptions)
CREATE OR REPLACE VIEW user_sub_details_vw AS
SELECT u.full_name, u.email, sp.plan_name, s.start_date, s.end_date, s.status
FROM users u
JOIN user_subscriptions s ON u.user_id = s.user_id
JOIN subscription_plans sp ON s.plan_id = sp.plan_id;

-- Fetch active subscribers (JOIN + WHERE – Users + Subscriptions)
SELECT u.full_name, s.subscription_id
FROM users u
JOIN user_subscriptions s ON u.user_id = s.user_id
WHERE s.status = 'ACTIVE';

-- Count users per plan (GROUP BY – Subscriptions)
select plan_id,count(user_id) as
total_users
from user_subscriptions
group by plan_id;

-- Show plans with more than 10 users (GROUP BY + HAVING – Subscriptions)
select plan_id,count(user_id) from user_subscriptions
group by plan_id
having count(user_id) > 10;

-- Check if subscription is active (Function – Subscriptions)
CREATE OR REPLACE FUNCTION check_sub_status (p_sub_id IN NUMBER) 
RETURN VARCHAR2 IS
    v_status VARCHAR2(20);
BEGIN
    SELECT status INTO v_status FROM user_subscriptions WHERE subscription_id = p_sub_id;
    RETURN v_status;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 'NOT_FOUND';
END;

-- 7. Watch History Module
-- Insert watch record (Stored Procedure – Watch_History table)
create or replace procedure add_watch_record
(p_history_id in int,p_user_id in int,p_content_id in int,p_watch_time in int)
is
begin
    insert into watch_history(history_id,user_id,content_id,watch_time,watched_at) values
    (p_history_id,p_user_id,p_content_id,p_watch_time,current_timestamp);
end;

-- Show watch history of user (JOIN – Users + Watch_History)
select u.full_name,w.content_id,w.watch_time,w.watched_at
from users u
join watch_history w on
u.user_id=w.user_id
where u.user_id=1;

-- Count total views per content (GROUP BY – Watch_History)
select content_id,count(*) as total_views
from watch_history
group by content_id;

-- Find most watched content (GROUP BY + ORDER BY – Watch_History)
select content_id,count(*) as views_count
from watch_history
group by content_id
order by views_count desc
fetch first 1 rows only;

-- Show user and content watched (JOIN – Watch_History + Content)
select u.full_name,c.title,w.watched_at from watch_history w
join users u on
w.user_id=u.user_id
join content c on
w.content_id=c.content_id;

-- 8. Payment Management Module
-- Record payment (Stored Procedure – Payments table)
CREATE OR REPLACE PROCEDURE record_payment (
    p_payment_id IN NUMBER,
    p_user_id IN NUMBER,
    p_sub_id IN NUMBER,
    p_amount IN NUMBER,
    p_method IN VARCHAR2
) AS
BEGIN
    INSERT INTO payments (payment_id, user_id, subscription_id, amount, payment_method, payment_status, payment_date)
    VALUES (p_payment_id, p_user_id, p_sub_id, p_amount, p_method, 'SUCCESS', CURRENT_TIMESTAMP);
    COMMIT;
END;

-- Fetch payment history (JOIN – Subscriptions + Payments)
select p.payment_id,p.amount,s.plan_id,p.payment_date from payments p
join user_subscriptions s on
p.subscription_id=s.subscription_id;

-- Calculate total revenue (Function SUM – Payments table)
create or replace function get_total_revenue
return number
is
    v_total number;
begin
    select nvl(sum(amount),0)
    into v_total
    from payments
    where payment_status='SUCCESS';

    return v_total;
end;


-- Calculate revenue per plan (JOIN + GROUP BY – Subscriptions + Payments)
create or replace view revenue_summary as
select s.plan_id,sum(p.amount) as total_revenue
from payments p
join user_subscriptions s on
p.subscription_id=s.subscription_id
where p.payment_status='SUCCESS'
group by s.plan_id;

-- Find users paying above average (Subquery – Payments table)
select user_id,amount from payments
where amount >(select avg(amount) from payments);
