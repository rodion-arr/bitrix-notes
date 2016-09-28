.. index::
single: Recipes for common tasks

Recipes for common tasks
========================

This section contains scripts for common tasks often found in work with Bitrix

Loop for all elements in iblock
-------------------------------
Getlist sorted by TIMESTAMP_X asc and limited by 1000 elements.

.. code-block:: php

    <?php
    CModule::IncludeModule('iblock');
    $arSelect = array('ID', 'NAME', 'CODE');
    $arFilter = array('IBLOCK_ID' => 10, 'ACTIVE' => 'Y');
    $res = CIBlockElement::GetList(
        array('TIMESTAMP_X' => 'ASC'),
        $arFilter,
        false,
        array('nPageSize' => 1000),
        $arSelect
    );
    while ($ob = $res->GetNextElement()) {
        $arFields = $ob->GetFields();

    }

Include section
---------------

.. code-block:: php

    <?$APPLICATION->IncludeFile(
         $APPLICATION->GetTemplatePath("/include/logo.php"),
         Array(),
         Array("MODE"=>"php")
    );?>

    <?$APPLICATION->IncludeComponent(
        "bitrix:main.include",
        "sidebar",
        Array(
            "AREA_FILE_SHOW" => "sect",
            "AREA_FILE_SUFFIX" => "inc",
            "AREA_FILE_RECURSIVE" => "N",
            "EDIT_MODE" => "html",
        ),
        false,
        Array('HIDE_ICONS' => 'Y')
    );?>

Menu _ext example
-----------------

.. code-block:: php

    <?php
    // .left.menu_ext.php

    if(!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true)die();

    global $APPLICATION;

    $aMenuLinksExt = $APPLICATION->IncludeComponent(
        "bitrix:menu.sections",
        "",
        array(
            "ID" => $_REQUEST["ELEMENT_ID"],
            "IBLOCK_TYPE" => "books",
            "IBLOCK_ID" => "30",
            "SECTION_URL" => "/e-store/books/index.php?SECTION_ID=#ID#",
            "CACHE_TIME" => "3600"
        )
    );

    $aMenuLinks = array_merge($aMenuLinks, $aMenuLinksExt);


Delete offers without photos
----------------------------

.. code-block:: php

    <?php
    CModule::IncludeModule('iblock');
    $arSelect = array('ID', 'DETAIL_PICTURE', 'PREVIEW_PICTURE');
    $arFilter = array('IBLOCK_ID' => 9, 'ACTIVE' => 'Y');
    $res = CIBlockElement::GetList(
        array('TIMESTAMP_X' => 'ASC'),
        $arFilter,
        false,
        array('nPageSize' => 1000),
        $arSelect
    );
    while ($ob = $res->GetNextElement()) {
        $arFields = $ob->GetFields();
        $bNoPics = false;

        if (empty($arFields['PREVIEW_PICTURE']) && empty($arFields['DETAIL_PICTURE'])) {
            $bNoPics = true;
        }

        if ($bNoPics) {
            $arSelect2 = array('ID', 'PROPERTY_CML2_LINK');
            $arFilter2 = array('IBLOCK_ID' => 13, 'PROPERTY_CML2_LINK' => $arFields['ID']);
            $res2 = CIBlockElement::GetList(
                array('TIMESTAMP_X' => 'ASC'),
                $arFilter2,
                false,
                array('nPageSize' => 1),
                $arSelect2
            );
            while ($ob2 = $res2->GetNextElement()) {
                $arFields2 = $ob2->GetFields();
                CIBlockElement::Delete($arFields2['ID']);
            }

        }
    }


Update all prices
-----------------

.. code-block:: php

    <?php
    CModule::IncludeModule('catalog');

    $db_res = CPrice::GetList(
        array(),
        array(
            'CURRENCY' => 'RUB'
        )
    );
    while ($ar_res = $db_res->Fetch()) {
        CPrice::Update($ar_res["ID"], ['CURRENCY' => 'UAH']);
    }

Detect mobile regex
-------------------

.. code-block:: php

    <?php
    $useragent=$_SERVER['HTTP_USER_AGENT'];
    if(preg_match('/android.+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|symbian|treo|up\.(browser|link)|vodafone|wap|windows (ce|phone)|xda|xiino/i',$useragent)||preg_match('/1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|e\-|e\/|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(di|rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|xda(\-|2|g)|yas\-|your|zeto|zte\-/i',substr($useragent,0,4))) {
        header('Location: http://m.site.ru '.$APPLICATION->GetCurPageParam());
    }

Build table with all products
-----------------------------

.. code-block:: php

    <?php
    require($_SERVER["DOCUMENT_ROOT"] . '/bitrix/modules/main/include/prolog_before.php');
    \Bitrix\Main\Loader::includeModule('iblock');

    $arSelect = array('ID', 'NAME', 'CODE', 'ACTIVE');
    $arFilter = array('IBLOCK_ID' => CATALOG_IBLOCK, 'ACTIVE' => 'Y');
    $res = CIBlockElement::GetList(
        array(),
        $arFilter,
        false,
        false,
        $arSelect
    );
    ?>
    <table>
        <tr>
            <td>ID</td>
            <td>Название</td>
            <td>Символьный код</td>
            <td>Активность</td>
        </tr>
        <?php while ($arFields = $res->GetNext()) {
            ?>
            <tr>
                <td><?= $arFields['ID'] ?></td>
                <td><?= $arFields['NAME'] ?></td>
                <td><?= $arFields['CODE'] ?></td>
                <td><?= $arFields['ACTIVE'] ?></td>
            </tr>
            <?php
        }
        ?>
    </table>

Read large CSV file
-------------------

.. code-block:: bash

    $ composer require league/csv:^8.0

.. code-block:: php

    <?php
    require_once '/vendor/autoload.php';
    use League\Csv\Reader;

    set_time_limit(0);
    ini_set('display_errors', 1);

    $importStep = 1000;
    $importStart = 0;

    $csv = Reader::createFromPath(__DIR__.'/your.csv');
    $csv->setDelimiter('|')
        ->setOffset($importStart)
        ->setLimit($importStep);

    $csv->each(function ($row, $rowOffset) {
        var_dump($row);

        return true;
    });

Cashing example
---------------

D7 style
~~~~~~~~

.. code-block:: php

    <?php
    use \Bitrix\Main\Data\Cache;

    $cache = Cache::createInstance();
    if ($cache->initCache(7200, "cache_key")) {
        $vars = $cache->getVars();
    }
    elseif ($cache->startDataCache()) {
        $cache->endDataCache(array("key" => "value"));
    }

    //public function initCache($TTL, $uniqueString, $initDir = false, $baseDir = "cache")

Managed cache

.. code-block:: php

    <?php
    $cache = \Bitrix\Main\Application::getInstance()->getManagedCache();
    
    if ($cache->read($cacheTtl, $cacheId)) {
        $vars = $cache->get($cacheId);
    } else {
        $cache->set($cacheId, array("key" => $value));
    }

    //clear by tag
    $cache->clean($cacheId);

Old style
~~~~~~~~~

.. code-block:: php

    <?php
    $cache = new CPHPCache;
    $cache_time = 3600;
    $cache_id = 'cache_id';
    if ($cache->InitCache($cache_time, $cache_id, '/' . SITE_ID . '/cache/path/')) {
        $arVars = $cache->GetVars();
        return $arVars['DATA'];
    } else {
        $cache->StartDataCache($cache_time, $cache_id);

        //get your data
        $data = 'some_text';

        if ($data) {
            $cache->EndDataCache(['DATA' => $data]);
            return $data;
        } else {
            return false;
        }
    }

Get Iblock id by it's code
--------------------------

.. code-block:: php

    <?php
    /**
     * Returns Iblock ID by it's code
     *
     * @param $iblockCode
     * @return bool
     */
    function getIblockIdByCode($iblockCode)
    {
        $cache = new CPHPCache;
        $cache_time = 3600;
        $cache_id = 'get_id_'.$iblockCode;
        if ($cache->InitCache($cache_time, $cache_id, '/'.SITE_ID.'/iblock/helper/')) {
            $arVars = $cache->GetVars();
            return $arVars['DATA']['ID'];
        } else {
            $cache->StartDataCache($cache_time, $cache_id);

            $arIblock = IblockTable::getList([
                'filter' => ['CODE' => $iblockCode],
                'select' => ['ID']
            ])->fetch();

            if ($arIblock) {
                $cache->EndDataCache(['DATA' => $arIblock]);
                return $arIblock['ID'];
            } else {
                return false;
            }
        }
    }

Add custom page in admin section
--------------------------------

Add new folder under your ``local`` folder

.. code-block:: bash

    $ mkdir local/bitrix.admin

Update ``urlrewrite.php`` file - add new rule

.. code-block:: php

    <?
    $arUrlRewrite = array(
        // ...
        array(
            'CONDITION' => '#^/bitrix/admin/(.*)#',
            'RULE' => '/local/bitrix.admin/$1',
            'ID' => '',
            'PATH' => '',
        ),
        // ...
    );

Admin page example. Place it in ``local/bitrix.admin`` folder

.. code-block:: php

    <?php
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/prolog_admin_before.php';
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/sale/include.php';

    /** @var CMain $APPLICATION */
    $saleModulePermissions = $APPLICATION->GetGroupRight('sale');

    /** @var CUser $USER */
    if (!$USER->IsAdmin()) {
        $APPLICATION->AuthForm(GetMessage('ACCESS_DENIED'));
    }

    IncludeModuleLangFile(__FILE__);

    $APPLICATION->SetTitle('Настройки eSputnik');

    require $_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/prolog_admin_after.php';
    ?>
    <?php
    $moduleName = 'esputnik';
    $settings = array(
        array(
            'LABEL' => 'Логин',
            'FORM_NAME' => 'esp-login',
            'OPTION_NAME' => 'LOGIN',
        ),
        array(
            'LABEL' => 'Пароль',
            'FORM_NAME' => 'esp-password',
            'TYPE' => 'password',
            'OPTION_NAME' => 'PASSWORD',
        ),
    );
    if (isset($_REQUEST['save']) && strlen($_REQUEST['save']) > 0) {
        foreach ($settings as $setting) {
            if (isset($_REQUEST[$setting['FORM_NAME']])) {
                COption::SetOptionString($moduleName, $setting['OPTION_NAME'], $_REQUEST[$setting['FORM_NAME']]);
            }
        }
    }
    ?>
        <style>
            .form-group {
                display: block;
                margin: 3px;
            }
            .form-group > label {
                display: inline-block;
                min-width: 200px;
            }
        </style>
        <form method="POST" action="<?= $APPLICATION->GetCurUri()?>">
            <?php foreach ($settings as $setting) :?>
                <div class="form-group" title="<?=$setting['DESCRIPTION']?>">
                    <label for="<?=$setting['FORM_NAME']?>"><?=$setting['LABEL']?>:</label>
                    <input type="<?=$setting['TYPE']?:'text'?>"
                           name="<?=$setting['FORM_NAME']?>" id="<?=$setting['FORM_NAME']?>"
                           value="<?=COption::getOptionString($moduleName, $setting['OPTION_NAME']);?>"
                           placeholder="<?=$setting['DESCRIPTION']?>"
                    >
                </div>
            <?php endforeach;?>
            <div class="form-group">
                <input type="submit" class="adm-btn" name="save" title="Сохранить" value="Сохранить">
            </div>
        </form>
    <?php
    require $_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/epilog_admin.php';

Add new admin menu item

.. code-block:: php

    AddEventHandler('main', 'OnBuildGlobalMenu', 'addMenuItem');

    function OnBuildGlobalMenuHandler(&$adminMenu, &$moduleMenu)
    {
        global $USER;
        if($USER->IsAdmin())
        {
            $moduleMenu[] = array(
                'parent_menu' => 'global_menu_store',
                'sort' => 10,
                'url' => 'your_new_page.php?lang='.LANG,
                'text' => 'your_new_page',
                'title' => 'your_new_page',
                'icon' => 'fav_menu_icon',
                'page_icon' => 'fav_menu_icon',
                'items_id' => 'menu_order',
            );
        }
    }

Set price extra for all products
--------------------------------

.. code-block:: php

    <?php
    function setGlobalPriceExtra()
    {
        $arSelect = ['ID', 'NAME'];
        $arFilter = ['IBLOCK_ID' => OFFERS_IBLOCK_ID,];
        $resOffer = CIBlockElement::GetList(
            [],
            $arFilter,
            false,
            ['nPageSize' => 5000],
            $arSelect
        );
        $el = new CIBlockElement;
        global $USER;

        \Bitrix\Main\Diag\Debug::dump($resOffer->SelectedRowsCount());
        \Bitrix\Main\Diag\Debug::dump('===========');

        while ($arFieldsOffer = $resOffer->GetNext()) {
            $arFields = array(
                "PRODUCT_ID" => $arFieldsOffer['ID'],
                "CATALOG_GROUP_ID" => 4,
                "EXTRA_ID" => 1,
                "CURRENCY" => "RUB",
            );

            $res = CPrice::GetList(
                array(),
                array(
                    "PRODUCT_ID" => $arFieldsOffer['ID'],
                    "CATALOG_GROUP_ID" => 4
                )
            );

            if ($arr = $res->Fetch()) {
                \Bitrix\Main\Diag\Debug::dump(CPrice::Update($arr["ID"], $arFields));
            } else {
                \Bitrix\Main\Diag\Debug::dump(CPrice::Add($arFields));
            }

            $arLoadProductArray = array(
                "MODIFIED_BY" => $USER->GetID(),
                "TIMESTAMP_X" => ConvertTimeStamp(time(), 'FULL')
            );
            $el->Update($arFieldsOffer['ID'], $arLoadProductArray);
        }
    }

Send instant message to users
-----------------------------

.. code-block:: php

    <?php
    CModule::IncludeModule("socialnetwork");
    $arFields = array(
        "FROM_USER_ID" => 2,
        "TO_USER_ID" => 1,
        "MESSAGE" => "Youк message",
        "=DATE_CREATE" => $GLOBALS["DB"]->CurrentTimeFunction(),
        "MESSAGE_TYPE" => "S",
    );
    CSocNetMessages::Add($arFields);
