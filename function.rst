.. index::
   single: Useful functions

Useful functions
================

This section contains useful functions

Dump for admin
--------------

.. code-block:: php

   <?php
   function pre($array, $exit = false, $hide = false)
   {
       global $USER;
       if ($USER->IsAdmin()) {
           if ($hide) {
               echo "<!--";
           }

           echo "<pre>";
           var_dump($array);
           echo "</pre>";

           if ($hide) {
               echo "-->";
           }

           if ($exit) {
               exit();
           }
       }
   }

Dump for admin limited by IP
----------------------------

.. code-block:: php

   <?php
   function pre($array, $exit = false, $hide = false)
   {
       if ($_SERVER["REMOTE_ADDR"] == "95.67.105.122") {
           if ($hide) {
               echo "<!--";
           }

           echo "<pre>";
           var_dump($array);
           echo "</pre>";

           if ($hide) {
               echo "-->";
           }

           if ($exit) {
               exit();
           }
       }
   }

Export array with short array syntax
------------------------------------

.. code-block:: php

   <?php
   /**
    * Var expor array as PHP 5.4 notation
    * @param        $var
    * @param string $indent
    * @return mixed|string
    */
   function var_export54($var, $indent = "")
   {
       switch (gettype($var)) {
           case "string":
               return '"' . addcslashes($var, "\\\$\"\r\n\t\v\f") . '"';
           case "array":
               $indexed = array_keys($var) === range(0, count($var) - 1);
               $r = [];
               foreach ($var as $key => $value) {
                   $r[] = "$indent    "
                       . ($indexed ? "" : var_export54($key) . " => ")
                       . var_export54($value, "$indent    ");
               }
               return "[\n" . implode(",\n", $r) . "\n" . $indent . "]";
           case "boolean":
               return $var ? "TRUE" : "FALSE";
           default:
               return var_export($var, TRUE);
       }
   }
