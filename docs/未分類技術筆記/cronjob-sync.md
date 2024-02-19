---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這一篇紀錄 cronjob-v2 搬家的流程
:::


## checkProductExpire - sync

這一篇是用同步的方式，但是用這種方式，去打每一個產品的連結，會非常的久！



```py
def set_old_db_product_expired():
    for EC_ID in EC_IDs:
        products = OldDBProduct.objects.filter(
            advertiser_id=EC_ID,
            expire__gte=timezone.now()
        )
        pks_has_expired = []
        data_sets = products.values_list('pk', 'link')
        logging.info('Check EC: {} total {} products'.format(EC_ID, data_sets.count()))
        for data in data_sets:
            resp = requests.get(data[1], allow_redirects=False)
            if resp.status_code != 200:
                pks_has_expired.append(data[0])

            time.sleep(2)    

        logging.info('EC: {} collect {} products to expire'.format(EC_ID, len(pks_has_expired)))                

        if pks_has_expired:            
            three_days_ago = timezone.now() - timezone.timedelta(days=3)
            products = OldDBProduct.objects.filter(advertiser_id=EC_ID, pk__in=pks_has_expired)
            products.update(live=False, expire=three_days_ago)



def set_new_db_product_expired():
    for EC_ID in EC_IDs:
        products = NewDBProduct.objects.filter(
            ec_id=EC_ID,
            expired_at__gte=timezone.now(),
        )
        pks_has_expired = []
        data_sets = products.values_list('pk', 'link')
        logging.info('Check EC: {} total {} products'.format(EC_ID, data_sets.count()))
        for data in data_sets:
            resp = requests.get(data[1], allow_redirects=False)
            if resp.status_code != 200:
                pks_has_expired.append(data[0])

        logging.info('EC: {} collect {} products to expire'.format(EC_ID, len(pks_has_expired)))

        if pks_has_expired:            
            three_days_ago = timezone.now() - timezone.timedelta(days=3)
            products = NewDBProduct.objects.filter(ec_id=EC_ID, pk__in=pks_has_expired)
            products.update(availability=Availability.OUT_OF_STOCK, expired_at=three_days_ago)


if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)

    set_old_db_product_expired()
    set_new_db_product_expired()
```



















