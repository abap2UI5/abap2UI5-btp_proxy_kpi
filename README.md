# abap2UI5-btp_proxy_kpi
🚧 work in progress 🚧
<br><br>
Find a way to send KPIs from ABAP to BTP. <br>
Maintain KPIs for abap2UI5 apps integrated in SAP Build Workzone Launchpad Service.
<br><br>
_Running into problems or found a bug? Create an issue [**here**](https://github.com/abap2UI5/abap2UI5/issues)_
<br>
<br>
#### Approach / First Idea:
(1/3) Use a single Interface:
```abap
INTERFACE z2ui5_if_proxy_kpi
  PUBLIC.

  METHODS count
    IMPORTING
      filter           TYPE string
    RETURNING
      VALUE(result) TYPE i.

ENDINTERFACE.
```
(2/3) Which can be used on app level to return KPIs:
```abap
CLASS z2ui5_cl_proxy_kpi_hello_world DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.
    INTERFACES z2ui5_if_proxy_kpi.
    INTERFACES z2ui5_if_app.

ENDCLASS.
CLASS z2ui5_cl_proxy_kpi_hello_world IMPLEMENTATION.

  METHOD z2ui5_if_proxy_kpi~count.

    "kpi calculation....
    result = 11.

  ENDMETHOD.

  METHOD z2ui5_if_app~main.
        "abap2UI5 app logic here...
  ENDMETHOD.

ENDCLASS.
```
(3/3) The rest handles a generic OData service (which just returns n dummy entries):
```abap
  METHOD /iwbep/if_mgw_appl_srv_runtime~get_entityset.

    DATA lt_result TYPE zcl_z2ui5_proxy_kpi_mpc=>tt_entity.
    DATA(lt_filter_cond) = io_tech_request_context->get_filter( )->get_filter_select_options( ).

    TRY.
        DATA(lv_classname)   = to_upper( lt_filter_cond[ property = `CLASS` ]-select_options[ 1 ]-low ).
      CATCH cx_root.
        INSERT VALUE #( id = `ERROR_NO_PARAMETER_FOUND_WITH_NAME_CLASS` ) INTO TABLE lt_result.
        copy_data_to_ref( EXPORTING is_data = lt_result CHANGING cr_data = er_entityset ).
        RETURN.
    ENDTRY.

    TRY.
        DATA(lv_filter) = to_upper( lt_filter_cond[ property = `FILTER` ]-select_options[ 1 ]-low ).
      CATCH cx_root.
    ENDTRY.

    DATA li_proxy_kpi TYPE REF TO z2ui5_if_proxy_kpi.
    CREATE OBJECT li_proxy_kpi TYPE (lv_classname).
    DATA(lv_count) = li_proxy_kpi->count( lv_filter ).

    DO lv_count TIMES.
      INSERT VALUE #( id = sy-index ) INTO TABLE lt_result.
    ENDDO.

    copy_data_to_ref( EXPORTING is_data = lt_result CHANGING cr_data = er_entityset ).

  ENDMETHOD.
```
