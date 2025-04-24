# Snowflake - The Definitive Guide

    - Snowsight, the new Snowflake web UI. 

    - After you log in, a client session is maintained indefinitely with continued user activity. After four hours of inactivity, the current session is terminated and you must log in again. The default session timeout policy of four hours can be changed; the minimum configurable idle timeout value for a session policy is five minutes.

    - To get the current role and warehouse
        ```sql
        SELECT CURRENT_ROLE();
        SELECT CURRENT_WAREHOUSE();
        ```
    
    - USE command to set the context for our database.

        ```sql
            USE DATABASE SNOWFLAKE_SAMPLE_DATA;
        ```