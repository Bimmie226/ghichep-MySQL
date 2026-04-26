# UPDATE DATA

| STT | Title                                                                                                     |
| --- | --------------------------------------------------------------------------------------------------------- |
| 1   | [UPDATE](https://github.com/Bimmie226/ghichep-MySQL/blob/master/BASICS/UPDATE%20DATA/UPDATE.md)           |
| 2   | [UPDATE JOIN](https://github.com/Bimmie226/ghichep-MySQL/blob/master/BASICS/UPDATE%20DATA/UPDATE_JOIN.md) |



START TRANSACTION;

UPDATE products
SET stock = stock - 1
WHERE id = 101 AND stock > 0;

INSERT INTO orders (user_id, total_amount, status)
VALUES (1, 500000, 'PAID');

INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (LAST_INSERT_ID(), 101, 1, 500000);

UPDATE payments
SET status = 'COMPLETED'
WHERE order_id = LAST_INSERT_ID();

COMMIT;