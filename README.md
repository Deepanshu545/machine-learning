# SUPBASE DATABASE 
---

 #### Abbreviation Map

### User / Role Abbreviations
- **AUAR** — Authenticated User (All Rows)
- **AUOR** — Authenticated User (Own Rows only)
- **SRAA** — Server Role (All Actions allowed)

### Action Abbreviations
- **S** — SELECT  
- **I** — INSERT  
- **U** — UPDATE  
- **D** — DELETE  

## Policies Summary

1. ms_closed_lots = S-AUOR , SRAA
2. ms_followers = I-AUOR , D-AUOR , S-AUOR
3. ms_instruments = S-AUAR , SRAA
4. ms_market_tags = S-AUAR
5. ms_open_lots = S-AUOR , SRAA
6. ms_portfolio_history = S-AUOR , SRAA
7. ms_stock_performance = S-AUAR
8. ms_trades = S-AUOR , SRAA
9. ms_user_details = S-AUOR , I-AUOR , U-AUOR , SRAA
10. ms_watchlist = S-AUOR , I-AUOR , U-AUOR , D-AUOR
11. users = S-AUOR , U-AUOR , SRAA

## GRANTS 
```

GRANT USAGE ON SCHEMA public TO authenticated;

-- -----------------------------
-- Table grants (authenticated)
-- -----------------------------
GRANT SELECT ON TABLE public.ms_closed_lots TO authenticated;
GRANT SELECT, INSERT, DELETE ON TABLE public.ms_followers TO authenticated;
GRANT SELECT, INSERT, UPDATE ON TABLE public.ms_user_details TO authenticated;
GRANT SELECT ON TABLE public.ms_market_tags TO authenticated;
GRANT SELECT ON TABLE public.ms_trades TO authenticated;
GRANT SELECT ON TABLE public.ms_portfolio_history TO authenticated;
GRANT SELECT ON TABLE public.ms_open_lots TO authenticated;
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE public.ms_watchlist TO authenticated;
GRANT SELECT, UPDATE ON TABLE public.users TO authenticated;
GRANT SELECT ON TABLE public.ms_instruments TO authenticated;
GRANT SELECT ON TABLE public.ms_stock_performances TO authenticated;

-- -----------------------------
-- Table grants (service_role)
-- -----------------------------
GRANT ALL ON TABLE public.ms_closed_lots TO service_role;
GRANT ALL ON TABLE public.ms_followers TO service_role;
GRANT ALL ON TABLE public.ms_user_details TO service_role;
GRANT ALL ON TABLE public.ms_market_tags TO service_role;
GRANT ALL ON TABLE public.ms_trades TO service_role;
GRANT ALL ON TABLE public.ms_portfolio_history TO service_role;
GRANT ALL ON TABLE public.ms_open_lots TO service_role;
GRANT ALL ON TABLE public.ms_watchlist TO service_role;
GRANT ALL ON TABLE public.users TO service_role;
GRANT ALL ON TABLE public.ms_instruments TO service_role;
GRANT ALL ON TABLE public.ms_stock_performances TO service_role;

-- -----------------------------
-- Sequence grants
-- (Only give auth. users sequence use where they INSERT directly)
-- -----------------------------
-- Authenticated needs sequence permission for tables they insert into:
GRANT USAGE, SELECT ON SEQUENCE public.ms_followers_id_seq TO authenticated;
GRANT USAGE, SELECT ON SEQUENCE public.ms_watchlist_id_seq TO authenticated;

-- Service_role should be able to use all sequences (backend writers)
GRANT ALL ON SEQUENCE public.ms_instruments_id_seq TO service_role;
GRANT ALL ON SEQUENCE public.ms_followers_id_seq TO service_role;
GRANT ALL ON SEQUENCE public.ms_trades_id_seq TO service_role;
GRANT ALL ON SEQUENCE public.ms_open_lots_id_seq TO service_role;
GRANT ALL ON SEQUENCE public.ms_closed_lots_id_seq TO service_role;
GRANT ALL ON SEQUENCE public.ms_watchlist_id_seq TO service_role;
GRANT ALL ON SEQUENCE public.ms_stock_performances_id_seq TO service_role;
GRANT ALL ON SEQUENCE public.ms_portfolio_history_id_seq TO service_role;

-- -----------------------------
-- End
-- -----------------------------

```



#  RPC Functions Overview
---
This section lists all RPC functions used in the system, along with their parameters and clear descriptions.

---

##### 1. `get_all_instruments_with_close_price()`
**Description:**  
Returns all instruments available in the market along with their latest close price.

---

##### 2. `get_leaderboard(p_period: string, p_limit: number, p_page: number)`
**Description:**  
Fetches paginated leaderboard data for a given time period.

**Parameters:**
- **p_period** — Leaderboard time window (e.g., `"1d"`, `"1w"`, `"1m"`).  
- **p_limit** — Number of records per page.  
- **p_page** — Page number.

---

##### 3. `get_trade_history(p_user_id: uuid, p_page: number, p_page_size: number)`
**Description:**  
Returns paginated trade history for the specified user.

---

##### 4. `update_user_profile(p_user_id: uuid, p_name: string)`
**Description:**  
Updates the user's profile (name and basic details).

---

##### 5. `add_to_watchlist(p_user_id: uuid, p_instrument_id: number)`
**Description:**  
Adds a specific instrument to the user’s watchlist.

---

##### 6. `get_watchlist(p_user_id: uuid)`
**Description:**  
Retrieves all instruments in the user’s watchlist.

---

##### 7. `remove_from_watchlist(p_user_id: uuid, p_instrument_id: number)`
**Description:**  
Removes a specific instrument from the user’s watchlist.

---

##### 8. `follow_user(p_follower_id: uuid, p_following_id: uuid)`
**Description:**  
Follows or unfollows another user.  
Calling it again with the same parameters toggles (unfollows).

---

##### 9. `get_instrument_details(p_instrument_id: number)`
**Description:**  
Returns complete details for a specific instrument.

---

##### 10. `get_user_profile_complete(p_user_id: uuid)`
**Description:**  
Retrieves the full public profile of a user.

---

##### 11. `get_user_portfolio_details(p_user_id: uuid)`
**Description:**  
Returns detailed portfolio metrics for the user.

---

##### 12. `search_instruments(p_search_term: string)`
**Description:**  
Searches for instruments by symbol or name.  
**Note:** Requires at least **2 characters**.

---


# SUPABASE SECRETS CREDENTIALS REQUIRED 
    1. GCP_MS_PUBLISHER_SA_KEY   
    2. LEGACY_JWT_SECRET           
    3. SUPABASE_ANON_KEY          
    4. SUPABASE_DB_URL             
    5. SUPABASE_SERVICE_ROLE_KEY   
    6. SUPABASE_URL  


 NOTE:  
  GCP_MS_PUBLISHER_SA_KEY  has permisions of firestore and pub sub topic , pub sub subscription 

---

# SUPABASE EDGE FUNCTIONS 
    1. order => edge function that takes order from frontend.
        `code in order-edge-function.ts`
    2. getPrice => get price of all stocks from firestore and call rpc function that calculate stock performance.

---

# GOOGLE RUN CREDENTIALS 
    code in `google-run/`

    - Has service role id which has access to firestore pub sub etc .
    - Pub sub has also permision to that particular id 

    - In ENVIRONMENTS variable 
        PULL_SUB	mockstock-orders-queue-topic-sub	
        BATCH_SIZE	1000	
        LOOP_SLEEP	2	
        COLLECTION_NAME	ticks	
        REQUEST_TIMEOUT	300

---

# Pub Sub 
    Has two subscription name 
        1. mockstock-orders-queue-sub-push (that push on google run )
        2. mockstock-orders-queue-topic-sub (that pull orders from google run )
    
    NOTE:- pull subscritpion has access to google run id to pull message.
    
    Has one topic name 
        1. mockstock-orders-queue-topic

    NOTE:- Topic and sub has permisions to alloe google run to pull message .

