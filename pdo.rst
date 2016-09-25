.. index::
   single: Example of using PDO

Example of using PDO
====================

This section contains scripts with PDO examples


.. code-block:: php

   <?php

   set_time_limit(0);
   ini_set('display_errors', 1);

   $time_start = microtime(true);

   $username = 'username';
   $password = 'password';
   $dsn = 'mysql:dbname=db_name;host=localhost';

   try {
       $conn = new PDO($dsn, $username, $password);
       $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

       //prepare example
       $insert = $conn->prepare("INSERT INTO table (col1, col2) VALUES (:col1, :col1)");
       $insert->bindParam(':col1', 'foo');
       $insert->bindParam(':col2', 'bar');
       $insert->execute();

       //last insert ID
       $conn->lastInsertId();

       //select query
       $select = $conn->query('SELECT * FROM table_1');
       $select->execute();
       $select->setFetchMode(PDO::FETCH_ASSOC);
       $result = $select->fetchAll();

   } catch (PDOException $e) {

   } catch (Exception $e) {

   }

   $conn = null;
