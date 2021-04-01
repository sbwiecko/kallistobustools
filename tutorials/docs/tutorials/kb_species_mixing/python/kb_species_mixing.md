# Pre-processing and analysis of mixed-species single-cell RNA-seq data with kallisto|bustools.

In this notebook, we will perform pre-processing and analysis of [10x Genomics 1k 1:1 mixure of fresh frozen human and mouse cells](https://support.10xgenomics.com/single-cell-gene-expression/datasets/3.0.2/1k_hgmm_v3) using the **kallisto | bustools** workflow, implemented with a wrapper called `kb`. It was developed by Kyung Hoi (Joseph) Min and A. Sina Booeshaghi.


```
!date
```

    Thu Jan 16 18:54:23 UTC 2020


## Pre-processing

### Download the data

__Note:__ We use the `-O` option for `wget` to rename the files to easily identify them.


```
%%time
!wget https://caltech.box.com/shared/static/8oeuskecfr9ujlufqj3b7frj74rxfzcc.txt -O checksums.txt
!wget https://caltech.box.com/shared/static/ags4jxbqrceuqewb0zy7kyuuggazqb0j.gz -O 1k_hgmm_v3_S1_L001_R1_001.fastq.gz
!wget https://caltech.box.com/shared/static/39tknal6wm4lhvozu6bf6vczb475bnuu.gz -O 1k_hgmm_v3_S1_L001_R2_001.fastq.gz
!wget https://caltech.box.com/shared/static/x2hwq2q3weuggtffjfgd1e8a1m1y7wj9.gz -O 1k_hgmm_v3_S1_L002_R1_001.fastq.gz
!wget https://caltech.box.com/shared/static/0g7lnuieg8jxlxswrssdtz809gus75ek.gz -O 1k_hgmm_v3_S1_L002_R2_001.fastq.gz
!wget https://caltech.box.com/shared/static/0avmybuxqcw8haa1hf0n72oyb8zriiuu.gz -O 1k_hgmm_v3_S1_L003_R1_001.fastq.gz
!wget https://caltech.box.com/shared/static/hp10z2yr8u3lbzoj1qflz83r2v9ohs6q.gz -O 1k_hgmm_v3_S1_L003_R2_001.fastq.gz
!wget https://caltech.box.com/shared/static/fx8fduedje53dvf3xixyyaqzugn7yy85.gz -O 1k_hgmm_v3_S1_L004_R1_001.fastq.gz
!wget https://caltech.box.com/shared/static/lpt6uzmueh1l2vx71nvsdj3pwqh8z3ak.gz -O 1k_hgmm_v3_S1_L004_R2_001.fastq.gz
```

    --2020-01-16 18:54:36--  https://caltech.box.com/shared/static/8oeuskecfr9ujlufqj3b7frj74rxfzcc.txt
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/8oeuskecfr9ujlufqj3b7frj74rxfzcc.txt [following]
    --2020-01-16 18:54:36--  https://caltech.box.com/public/static/8oeuskecfr9ujlufqj3b7frj74rxfzcc.txt
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/8oeuskecfr9ujlufqj3b7frj74rxfzcc.txt [following]
    --2020-01-16 18:54:36--  https://caltech.app.box.com/public/static/8oeuskecfr9ujlufqj3b7frj74rxfzcc.txt
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!bfmyuKChl9mlNrvM6NKR0M68SdDjLgsYPYDbhlX2FmS_ZXAPPd_pMhS3ctm-xienppt-TjTuthbN2Z0NFzY94kGDhyUbFSAeMBhJMo1RFQHlD-_pCmKgyFLg5jYM7fBGFWYC-PpKwd6Pna0Y6RwuNiHyTUZKJbep5tzrRhbXrlDEVRxwC6p3RcjJFo4GChO3XsdUhh_y8f7P6AAwynNro6pZPX2Vk8GbEsBwJi8Z9kfZNZAjk0cOFFLDgq0xAdnVH-_tT41ILHT7ANRV2jFLJM6vQCwHrUcfw1oB5O0pBRcuF8_X9EiMT1a60aRFo4HJRtinBP4-Lc41oeOAcH2Q59666iK0NqrOZl2lnmF0xcrcpEectHpff-jg8AEZsqJ_4CdVQQnXVluT_w5kglccQYIpXDfkyGBB94Ib_v6wtBsXzwafxYg35z-RRk6XWYaRcx49SnxJNjAKlIdFOHcAmnfK-H7tk4QN1yW5V_fFe169MyBc2GShaX3ZOTAHW5HyRCscM0VycqQFhfN62ak3cXjxE6f-FA4l4qLqoCp5OsLQW6i-r_fefmo92TlnPHVHbUYZC-cCSfjZwEZiM0rnHnTDG_QacLNOMiicE6qYSQZ1cv2IpWWlzgTg9NFrun-cmjcne3kOvRQCVuhuiXb7yXCs8uoBJzDijBfepy6DF6TDkVFDANLdkkgO_8QziwmHKNsPLg7P-RF262lmtRnWtwvhW8XU9JFu0IJmnkG-UGeB4FBGxN28eRbAUxWzLm-KQe_rj-Ym0hRXnv7dg5zo_M2tQX6DK32pDHAaaqE832ta_iN0-AR82yPDSElVA1VGOsHgjzjfeqPTvRuFnu5pf6aBHI5I1-ppLMXKKQVauHJKKwV3dOBPrdGA2VLFVGVLgrX7FX1K7rzHk_tU7OUawhYHZO_Y44recX1svQLbFcX5KFp4MjfLaOI4W0z4gosUOH9lrb-XU23AQXhkKawuBk0Xf-6P_fsICcrguBvm2t41KpejEcJAwPl2JeKurRHIObdnuyXa9pPGhODme0T8HvNL9grmJXCV8BJfEQs0wDcc2NUWdftsLqh5ASwAncaIkflEELx0iC7T0V1zmMenqTVGkQUajYql4weAoUwejK4ELE07fYaXmed68yZvT_GEia5yj6i05udh6W_NcycQbmKsbHuI9rUhuiwjI3S1RlNxWCO9E-rs_WJoc20thLlDOGZ4XC0zTwRkaX19OK8A9qiKv48gUaqpHJlWBiRjQND7iSBG6rNU4oqF9105Mqi0rn1zrwxYtYIp-2OaF_PKqDLdzd8RZLO5_PfW1280fK45Xn2eUpioK48qZIIMuDP0En_91QnD_13RTFfM7hqU15BToeqDV9IIsNJe/download [following]
    --2020-01-16 18:54:37--  https://public.boxcloud.com/d/1/b1!bfmyuKChl9mlNrvM6NKR0M68SdDjLgsYPYDbhlX2FmS_ZXAPPd_pMhS3ctm-xienppt-TjTuthbN2Z0NFzY94kGDhyUbFSAeMBhJMo1RFQHlD-_pCmKgyFLg5jYM7fBGFWYC-PpKwd6Pna0Y6RwuNiHyTUZKJbep5tzrRhbXrlDEVRxwC6p3RcjJFo4GChO3XsdUhh_y8f7P6AAwynNro6pZPX2Vk8GbEsBwJi8Z9kfZNZAjk0cOFFLDgq0xAdnVH-_tT41ILHT7ANRV2jFLJM6vQCwHrUcfw1oB5O0pBRcuF8_X9EiMT1a60aRFo4HJRtinBP4-Lc41oeOAcH2Q59666iK0NqrOZl2lnmF0xcrcpEectHpff-jg8AEZsqJ_4CdVQQnXVluT_w5kglccQYIpXDfkyGBB94Ib_v6wtBsXzwafxYg35z-RRk6XWYaRcx49SnxJNjAKlIdFOHcAmnfK-H7tk4QN1yW5V_fFe169MyBc2GShaX3ZOTAHW5HyRCscM0VycqQFhfN62ak3cXjxE6f-FA4l4qLqoCp5OsLQW6i-r_fefmo92TlnPHVHbUYZC-cCSfjZwEZiM0rnHnTDG_QacLNOMiicE6qYSQZ1cv2IpWWlzgTg9NFrun-cmjcne3kOvRQCVuhuiXb7yXCs8uoBJzDijBfepy6DF6TDkVFDANLdkkgO_8QziwmHKNsPLg7P-RF262lmtRnWtwvhW8XU9JFu0IJmnkG-UGeB4FBGxN28eRbAUxWzLm-KQe_rj-Ym0hRXnv7dg5zo_M2tQX6DK32pDHAaaqE832ta_iN0-AR82yPDSElVA1VGOsHgjzjfeqPTvRuFnu5pf6aBHI5I1-ppLMXKKQVauHJKKwV3dOBPrdGA2VLFVGVLgrX7FX1K7rzHk_tU7OUawhYHZO_Y44recX1svQLbFcX5KFp4MjfLaOI4W0z4gosUOH9lrb-XU23AQXhkKawuBk0Xf-6P_fsICcrguBvm2t41KpejEcJAwPl2JeKurRHIObdnuyXa9pPGhODme0T8HvNL9grmJXCV8BJfEQs0wDcc2NUWdftsLqh5ASwAncaIkflEELx0iC7T0V1zmMenqTVGkQUajYql4weAoUwejK4ELE07fYaXmed68yZvT_GEia5yj6i05udh6W_NcycQbmKsbHuI9rUhuiwjI3S1RlNxWCO9E-rs_WJoc20thLlDOGZ4XC0zTwRkaX19OK8A9qiKv48gUaqpHJlWBiRjQND7iSBG6rNU4oqF9105Mqi0rn1zrwxYtYIp-2OaF_PKqDLdzd8RZLO5_PfW1280fK45Xn2eUpioK48qZIIMuDP0En_91QnD_13RTFfM7hqU15BToeqDV9IIsNJe/download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 816 [text/plain]
    Saving to: ‘checksums.txt’
    
    checksums.txt       100%[===================>]     816  --.-KB/s    in 0s      
    
    2020-01-16 18:54:38 (89.8 MB/s) - ‘checksums.txt’ saved [816/816]
    
    --2020-01-16 18:54:39--  https://caltech.box.com/shared/static/ags4jxbqrceuqewb0zy7kyuuggazqb0j.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/ags4jxbqrceuqewb0zy7kyuuggazqb0j.gz [following]
    --2020-01-16 18:54:39--  https://caltech.box.com/public/static/ags4jxbqrceuqewb0zy7kyuuggazqb0j.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/ags4jxbqrceuqewb0zy7kyuuggazqb0j.gz [following]
    --2020-01-16 18:54:39--  https://caltech.app.box.com/public/static/ags4jxbqrceuqewb0zy7kyuuggazqb0j.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!oAVoFP4CGR1yzS3AhdhiaAdMAqPE05UNEXs7urCZBMlXrLO-eC_WoLKu2DicVApzVB9uFBwPyVG6qVvLSuO2q5RhwOFJXQEue5oTPrDX1Oelxh_0A28enjR8a7iNaqt2ez03RXB_1h0zIVpGjGSU6TRlA-x335MOdkAAggJljV_2P7TsmgUNHxPiA11T3Hfg-epxmbm0r-0dWrivQy5LVfIGMidRAgT6HtNUw2lhYJEDPBIJ97sN_Amcd8EaWdialvo4aARr9yyRptTpd9fbZe6n5yDW8DBL6PE-VKRFGKXHUi_RHseIWha-fuCnBFPujgex1nWiI4fvDDm3ZGgH7uzQTTnKItBddTOqcjZqGhAlyMXw4f4FMJyKTYgud4SudpKwPzRO0-dI8YduzmDvbwTynwAqNq2tMiQ3tRfXp9sc7M3ePoKG1e2SuYipRCfb_i2CAb2p7hH8MwKFE9KyAPh44LJ6fZI4GP8bYEy92_K0qcSR1bm0T5FufAGP-Z8dmjX93Ck08JSed15BwlE2tT7nlqfCU1OeTyxHjxS7zDpTqAehjxuJyhfo8Yrc6z2Do8Yr_25MyEW_Cn7XBYYGqvcGQ0Cw2j2UQvtsxNstEQP3trK_PP44zUMmnewU269KOG0jLJvi3f65HkaOQf_o775BLbrZsVcOwg82qJ5uR3w-QQy50FHIag0bZJPy6dWqvvorsMDeFt5rrp-NdPlr818AoA4yti-Rxf63WQv1z0yrU2YeYNSNTeQZvAcGbhUJqepo6ohC9ogv5zgRBQF21l0GfDggPtV-gO-QMPh64LGho922p8KyLEDXRUg_8MLR-ZnjUs2C3Cdpw9Dnj1XtbH_EBg3efe_AYXV6UtgUb_AQIMjrD2L7W349IHfS5-WKsuSPgx9go4Oxi9uCe0ydrH9157AZryUKvj1esq-O6CAYHTUpNSo1VW37VgcZxuKnLkBtvs82TVER9_cIwn0dT8vuOkCnKyYTwZbcWi1oAYimjpA7VuLh4vcFlAMZO0aq_Hni4QY0v4Z7JzcYxmcRTvbDJWsnH6EG7bUAkaLO90tyQ-cZcCVQQDpLRiX0EewF-tsqGqBLZPwhJ9uIrEysE85Epd3KzS59SuF7TIvaF7FD9uwKHZsg-MsZDMsXU4ms8SQvCmE7a43yQJHqJgO5zF38bYW_SVk_3qfcLLixG_3ALPapnYf4VFAob9c34TBKmaWLiLteSE-hPsMaLYABMf9PoCh2T9hlidzqvBrYP8RPPo9N4w-yrqidTRo-r1EGCXcBF9CfqeHSxaviQYa9F50dbqUBPXAA4gl4nxmZppjtFuidX3nx5dMkwLcnNriaE4S4S39XWyIsjo7SHWPWJEJp6c5goE6VOI8sP2-LXKd7jMgl9iXCz-69nFWAvDCsCybxBxdMdek0t_qC0lYX3PYsN_B-/download [following]
    --2020-01-16 18:54:40--  https://public.boxcloud.com/d/1/b1!oAVoFP4CGR1yzS3AhdhiaAdMAqPE05UNEXs7urCZBMlXrLO-eC_WoLKu2DicVApzVB9uFBwPyVG6qVvLSuO2q5RhwOFJXQEue5oTPrDX1Oelxh_0A28enjR8a7iNaqt2ez03RXB_1h0zIVpGjGSU6TRlA-x335MOdkAAggJljV_2P7TsmgUNHxPiA11T3Hfg-epxmbm0r-0dWrivQy5LVfIGMidRAgT6HtNUw2lhYJEDPBIJ97sN_Amcd8EaWdialvo4aARr9yyRptTpd9fbZe6n5yDW8DBL6PE-VKRFGKXHUi_RHseIWha-fuCnBFPujgex1nWiI4fvDDm3ZGgH7uzQTTnKItBddTOqcjZqGhAlyMXw4f4FMJyKTYgud4SudpKwPzRO0-dI8YduzmDvbwTynwAqNq2tMiQ3tRfXp9sc7M3ePoKG1e2SuYipRCfb_i2CAb2p7hH8MwKFE9KyAPh44LJ6fZI4GP8bYEy92_K0qcSR1bm0T5FufAGP-Z8dmjX93Ck08JSed15BwlE2tT7nlqfCU1OeTyxHjxS7zDpTqAehjxuJyhfo8Yrc6z2Do8Yr_25MyEW_Cn7XBYYGqvcGQ0Cw2j2UQvtsxNstEQP3trK_PP44zUMmnewU269KOG0jLJvi3f65HkaOQf_o775BLbrZsVcOwg82qJ5uR3w-QQy50FHIag0bZJPy6dWqvvorsMDeFt5rrp-NdPlr818AoA4yti-Rxf63WQv1z0yrU2YeYNSNTeQZvAcGbhUJqepo6ohC9ogv5zgRBQF21l0GfDggPtV-gO-QMPh64LGho922p8KyLEDXRUg_8MLR-ZnjUs2C3Cdpw9Dnj1XtbH_EBg3efe_AYXV6UtgUb_AQIMjrD2L7W349IHfS5-WKsuSPgx9go4Oxi9uCe0ydrH9157AZryUKvj1esq-O6CAYHTUpNSo1VW37VgcZxuKnLkBtvs82TVER9_cIwn0dT8vuOkCnKyYTwZbcWi1oAYimjpA7VuLh4vcFlAMZO0aq_Hni4QY0v4Z7JzcYxmcRTvbDJWsnH6EG7bUAkaLO90tyQ-cZcCVQQDpLRiX0EewF-tsqGqBLZPwhJ9uIrEysE85Epd3KzS59SuF7TIvaF7FD9uwKHZsg-MsZDMsXU4ms8SQvCmE7a43yQJHqJgO5zF38bYW_SVk_3qfcLLixG_3ALPapnYf4VFAob9c34TBKmaWLiLteSE-hPsMaLYABMf9PoCh2T9hlidzqvBrYP8RPPo9N4w-yrqidTRo-r1EGCXcBF9CfqeHSxaviQYa9F50dbqUBPXAA4gl4nxmZppjtFuidX3nx5dMkwLcnNriaE4S4S39XWyIsjo7SHWPWJEJp6c5goE6VOI8sP2-LXKd7jMgl9iXCz-69nFWAvDCsCybxBxdMdek0t_qC0lYX3PYsN_B-/download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 419837628 (400M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L001_R1_001.fastq.gz’
    
    1k_hgmm_v3_S1_L001_ 100%[===================>] 400.39M  16.4MB/s    in 25s     
    
    2020-01-16 18:55:06 (16.0 MB/s) - ‘1k_hgmm_v3_S1_L001_R1_001.fastq.gz’ saved [419837628/419837628]
    
    --2020-01-16 18:55:07--  https://caltech.box.com/shared/static/39tknal6wm4lhvozu6bf6vczb475bnuu.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/39tknal6wm4lhvozu6bf6vczb475bnuu.gz [following]
    --2020-01-16 18:55:08--  https://caltech.box.com/public/static/39tknal6wm4lhvozu6bf6vczb475bnuu.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/39tknal6wm4lhvozu6bf6vczb475bnuu.gz [following]
    --2020-01-16 18:55:08--  https://caltech.app.box.com/public/static/39tknal6wm4lhvozu6bf6vczb475bnuu.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!eMeEHUzOmOGHx4f5b3u4YGvAAT_ZvAOou4Cdq-FhcruaF75WIj_4ktPNtpHE9MhcjGV9Zf4k8vUAmELv9i8U4ZYKMGd2ThQpny70Iw9WTklsVgfJ3F5dpQCA-MDosYM72KQHxse5bPCuG_4TUqfGDj3jntaMzVgv9aRnMBVf2a9Brzq4NZJGImlNQnIN9q4ZhKgfuFrrU5qoUSRJO2snYBdWw9VNf5EuXi3ZrlUWw7JKSMho_DYuM-s5REZZj_YMykt7l2wlY2E490vAeXj7FxU2P2heWW6uVLyfdDrGfFDpaB4XYIezd2KXHauKm8WQxZ1Kk9Mm-HGkRV9sqen_lvgzcu4o1dWiThdiIYN_ayiEqPIEsKef-NKnXn-LY1QpVR716vhuEX3vUkoSfddWiuG2cRC8RF_aLyRFEbLjk30oPbgP-aXhqWIuXaHHHfUJ9NoRetWsfkCG3bMWNOJn_OA1RP52tUKD9R7dhdiRkp83aCTTcod2pSR9VBOUvWNcb2b5X0HlR4Y-zyRo-ZXTL0QQZENrSzOCYUoFB-3RrepdxTjIhsiZte0mJKRA8TD68gR3dmcAv8kWB7e9gYdrSYleCl6xv_zhgIJSRQgmmB87KAmjepi9vSRflAVbLSvwzzPqIDroA5-t2DnOt2mCCKQUusghM37RkwIY4l-2s8iRntB5JE-QGZdemCWl8PivZ5S8hE6toPtR9mHlZn8fYX9DaB5HtMnzP7McFIg0iFtuVW7PP6_yYDHxXpoFAVvvNDQ1BahrO3pTzjy-1nZobsjleqDdJkwvybDCOm8BAml4WQ1e-Gl5fFieTQEBNySygOT-uhA4XQvdYbz1okKHO5DASspk-UxRhOg81ySuJlr__6WOMd0xGEF7EgUDQCBYs_V8MPhy_-6l4v7YkREDQAmKRrSUnrTYaG2MBZg9VH3F5mwh4UHf0us5zWXjNNdapZgD10ea9b6t7m9yRqxlOgeIHiV5KYuKuvti9IeF9zFNSHSUj5h0_zIdOSvf5EX54K-xinnIMh0fpdeZQOUkmoLX5EqJV_HBgxBTb0zCRXhxmrxplYkNfsOS2Bpb9QANXFIm49G__tjdP7ad-mIPVOEh7Vz3AFcS5f7eAvOx4zHCWsivQ9kCCUyKpuHWDC27EHtM_MTJzugZLCLZuhAC0V355fohXfa09QGr34l2nGJQ3WrnuJBMtmIDzYNFdT7iCyEpf_buJErlZqLVzIx4AxQKUrPXGSWgHB_tvELrfcYkoi53M9CAUJcEvou2KksxShc1GGTldmYrswP1j5_HIL21thqJYIE8KjZ4AoCd2WWJ4VnW3g53shEeWeUThpKue_cpDVujhQVBoWRa_-HZ0xjEGHGDt4MD6SaQSlZklLaKgzGLmXvkp-U7Mg5Wry6LTtLo30xffseOxOx5QXCFw6swPyI./download [following]
    --2020-01-16 18:55:09--  https://public.boxcloud.com/d/1/b1!eMeEHUzOmOGHx4f5b3u4YGvAAT_ZvAOou4Cdq-FhcruaF75WIj_4ktPNtpHE9MhcjGV9Zf4k8vUAmELv9i8U4ZYKMGd2ThQpny70Iw9WTklsVgfJ3F5dpQCA-MDosYM72KQHxse5bPCuG_4TUqfGDj3jntaMzVgv9aRnMBVf2a9Brzq4NZJGImlNQnIN9q4ZhKgfuFrrU5qoUSRJO2snYBdWw9VNf5EuXi3ZrlUWw7JKSMho_DYuM-s5REZZj_YMykt7l2wlY2E490vAeXj7FxU2P2heWW6uVLyfdDrGfFDpaB4XYIezd2KXHauKm8WQxZ1Kk9Mm-HGkRV9sqen_lvgzcu4o1dWiThdiIYN_ayiEqPIEsKef-NKnXn-LY1QpVR716vhuEX3vUkoSfddWiuG2cRC8RF_aLyRFEbLjk30oPbgP-aXhqWIuXaHHHfUJ9NoRetWsfkCG3bMWNOJn_OA1RP52tUKD9R7dhdiRkp83aCTTcod2pSR9VBOUvWNcb2b5X0HlR4Y-zyRo-ZXTL0QQZENrSzOCYUoFB-3RrepdxTjIhsiZte0mJKRA8TD68gR3dmcAv8kWB7e9gYdrSYleCl6xv_zhgIJSRQgmmB87KAmjepi9vSRflAVbLSvwzzPqIDroA5-t2DnOt2mCCKQUusghM37RkwIY4l-2s8iRntB5JE-QGZdemCWl8PivZ5S8hE6toPtR9mHlZn8fYX9DaB5HtMnzP7McFIg0iFtuVW7PP6_yYDHxXpoFAVvvNDQ1BahrO3pTzjy-1nZobsjleqDdJkwvybDCOm8BAml4WQ1e-Gl5fFieTQEBNySygOT-uhA4XQvdYbz1okKHO5DASspk-UxRhOg81ySuJlr__6WOMd0xGEF7EgUDQCBYs_V8MPhy_-6l4v7YkREDQAmKRrSUnrTYaG2MBZg9VH3F5mwh4UHf0us5zWXjNNdapZgD10ea9b6t7m9yRqxlOgeIHiV5KYuKuvti9IeF9zFNSHSUj5h0_zIdOSvf5EX54K-xinnIMh0fpdeZQOUkmoLX5EqJV_HBgxBTb0zCRXhxmrxplYkNfsOS2Bpb9QANXFIm49G__tjdP7ad-mIPVOEh7Vz3AFcS5f7eAvOx4zHCWsivQ9kCCUyKpuHWDC27EHtM_MTJzugZLCLZuhAC0V355fohXfa09QGr34l2nGJQ3WrnuJBMtmIDzYNFdT7iCyEpf_buJErlZqLVzIx4AxQKUrPXGSWgHB_tvELrfcYkoi53M9CAUJcEvou2KksxShc1GGTldmYrswP1j5_HIL21thqJYIE8KjZ4AoCd2WWJ4VnW3g53shEeWeUThpKue_cpDVujhQVBoWRa_-HZ0xjEGHGDt4MD6SaQSlZklLaKgzGLmXvkp-U7Mg5Wry6LTtLo30xffseOxOx5QXCFw6swPyI./download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 968233265 (923M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L001_R2_001.fastq.gz’
    
    1k_hgmm_v3_S1_L001_ 100%[===================>] 923.38M  15.9MB/s    in 60s     
    
    2020-01-16 18:56:09 (15.4 MB/s) - ‘1k_hgmm_v3_S1_L001_R2_001.fastq.gz’ saved [968233265/968233265]
    
    --2020-01-16 18:56:11--  https://caltech.box.com/shared/static/x2hwq2q3weuggtffjfgd1e8a1m1y7wj9.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/x2hwq2q3weuggtffjfgd1e8a1m1y7wj9.gz [following]
    --2020-01-16 18:56:11--  https://caltech.box.com/public/static/x2hwq2q3weuggtffjfgd1e8a1m1y7wj9.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/x2hwq2q3weuggtffjfgd1e8a1m1y7wj9.gz [following]
    --2020-01-16 18:56:11--  https://caltech.app.box.com/public/static/x2hwq2q3weuggtffjfgd1e8a1m1y7wj9.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!1NvTdn9tUiR6SLQjou9XPBfP3IARW-Zvl-rfuaQwKGeKpakXmc_T1ALgvc8tt7nwNO5GQLgTmRLsnBzIvu1J7NeuOlb1hHe0W-xh3JFBNjG-bbl-2dfivsp6nYJhqlkzp3QwvDDGqfUBNIpviNYqBmgudZ4cBacwqGgS76XM4y7E7P9ZrE2taHquPWtTQeyYh-uJtNLFys4HEXOK79bMz1z9ToTAA_5tz_5gxYUzVv3gdy13lO2kt3_ZzEIPXmBUK4b6GRRpqjH7eVKj_GJR_UMCw-hCmAKLoKM099ZSHzaOdBFH8M4SxwKfms9MG_nX98wXtqpeaKviS7Fn67aPQdU00KQ6JAwsgZs7ATYS7Xf9nVvcSX133cvbRtPsmn-9k8O5Pk7xDV1lN_GdkzA56QOw72NX-tbKtkD_X8-4hxiEPLIj51OITqEMDNoLSu4-1e0MBmNcH83fShvUBUXDGfgE5k8OyNpuw7EKxgipU6mrZQwvvsfxBGrBy37f_WB0kcgrojNbP4N6UZXJIJlzhaH4lqY2Eoj2yfYkQ1YywsEvvN7nKtr7ByGcwZrhxDLzHDGlNpTyYeIf0uDaqjcgGmAUhwv_KZfAn31Fa7zhjq2tgR7vsdJV3dIYVoIwktid5tau2slGnS2KIF72i0zXeN3kdudKaM6t27W0fV4lBd8kG41EYZTd86ollhJ6hvAktgeDwSqvawzdL7me_SkPbhoEoT1dMiF3-yxhyVwen17QxGETLH4wbmhJufBcr5umliRDi3yAquHiGVrqTboo5O7ezNIxuhjm6ExOH7OcHaxke2cJxv_faC-WHt8BWoh64ABbm_mRYAjY3AssacpRQ_VIrzGLWI_TRNWDRmHv7pPyS_JUTQTBT40OHcB4_nFtdQPZe7SnmEUcvq1YeXU6BF9NeV7BiG6kFFrWH-4tUSMN_u6XEGo_MG7uAWv9M6aHAUqjTY1UxwQPMVAsakQTjAKMsWg0LgdJTJ8zs7FW-uTgwFEbDVXgY18HYrHv61hhPmchK2cUIqi6p6JJ5Stir7LesvhtFaVWWeurSGvRDp6fFiDbBvPNzUWH7YI_SnyUVpC0ugLUR2RNAYBTgAgRO-HJqPASe1CEV0pE1zLs70-Gn4EOsGWfYoYlqKfpshctoOi4-NdEStPP-fpBVj0uOO604RRzb-L75ZVg9LRGaOXOWelLN6SE_b_W3aOrveb8xe7A2TU_7zX4lt1fEHthCBjCaT9fCOSWFdcIJRD4uYVJnKziwYh3ZR9pATvVIyVUUxMlgoIYTkId6wMcvSELUZ2yRD2YXmnbwOH889IEiBX-o4_HRxjuvMUqHLXaIVsvA8VvOh5XQCclwif5E3zIoXSXiYlLqdTEAJGN9lNz7xK_GZhiB0fnsgdWdyEcMvKEQWh1WjnFqO8ySAQFtGJlROErNP8./download [following]
    --2020-01-16 18:56:12--  https://public.boxcloud.com/d/1/b1!1NvTdn9tUiR6SLQjou9XPBfP3IARW-Zvl-rfuaQwKGeKpakXmc_T1ALgvc8tt7nwNO5GQLgTmRLsnBzIvu1J7NeuOlb1hHe0W-xh3JFBNjG-bbl-2dfivsp6nYJhqlkzp3QwvDDGqfUBNIpviNYqBmgudZ4cBacwqGgS76XM4y7E7P9ZrE2taHquPWtTQeyYh-uJtNLFys4HEXOK79bMz1z9ToTAA_5tz_5gxYUzVv3gdy13lO2kt3_ZzEIPXmBUK4b6GRRpqjH7eVKj_GJR_UMCw-hCmAKLoKM099ZSHzaOdBFH8M4SxwKfms9MG_nX98wXtqpeaKviS7Fn67aPQdU00KQ6JAwsgZs7ATYS7Xf9nVvcSX133cvbRtPsmn-9k8O5Pk7xDV1lN_GdkzA56QOw72NX-tbKtkD_X8-4hxiEPLIj51OITqEMDNoLSu4-1e0MBmNcH83fShvUBUXDGfgE5k8OyNpuw7EKxgipU6mrZQwvvsfxBGrBy37f_WB0kcgrojNbP4N6UZXJIJlzhaH4lqY2Eoj2yfYkQ1YywsEvvN7nKtr7ByGcwZrhxDLzHDGlNpTyYeIf0uDaqjcgGmAUhwv_KZfAn31Fa7zhjq2tgR7vsdJV3dIYVoIwktid5tau2slGnS2KIF72i0zXeN3kdudKaM6t27W0fV4lBd8kG41EYZTd86ollhJ6hvAktgeDwSqvawzdL7me_SkPbhoEoT1dMiF3-yxhyVwen17QxGETLH4wbmhJufBcr5umliRDi3yAquHiGVrqTboo5O7ezNIxuhjm6ExOH7OcHaxke2cJxv_faC-WHt8BWoh64ABbm_mRYAjY3AssacpRQ_VIrzGLWI_TRNWDRmHv7pPyS_JUTQTBT40OHcB4_nFtdQPZe7SnmEUcvq1YeXU6BF9NeV7BiG6kFFrWH-4tUSMN_u6XEGo_MG7uAWv9M6aHAUqjTY1UxwQPMVAsakQTjAKMsWg0LgdJTJ8zs7FW-uTgwFEbDVXgY18HYrHv61hhPmchK2cUIqi6p6JJ5Stir7LesvhtFaVWWeurSGvRDp6fFiDbBvPNzUWH7YI_SnyUVpC0ugLUR2RNAYBTgAgRO-HJqPASe1CEV0pE1zLs70-Gn4EOsGWfYoYlqKfpshctoOi4-NdEStPP-fpBVj0uOO604RRzb-L75ZVg9LRGaOXOWelLN6SE_b_W3aOrveb8xe7A2TU_7zX4lt1fEHthCBjCaT9fCOSWFdcIJRD4uYVJnKziwYh3ZR9pATvVIyVUUxMlgoIYTkId6wMcvSELUZ2yRD2YXmnbwOH889IEiBX-o4_HRxjuvMUqHLXaIVsvA8VvOh5XQCclwif5E3zIoXSXiYlLqdTEAJGN9lNz7xK_GZhiB0fnsgdWdyEcMvKEQWh1WjnFqO8ySAQFtGJlROErNP8./download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 423258946 (404M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L002_R1_001.fastq.gz’
    
    1k_hgmm_v3_S1_L002_ 100%[===================>] 403.65M  15.1MB/s    in 27s     
    
    2020-01-16 18:56:40 (14.7 MB/s) - ‘1k_hgmm_v3_S1_L002_R1_001.fastq.gz’ saved [423258946/423258946]
    
    --2020-01-16 18:56:42--  https://caltech.box.com/shared/static/0g7lnuieg8jxlxswrssdtz809gus75ek.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/0g7lnuieg8jxlxswrssdtz809gus75ek.gz [following]
    --2020-01-16 18:56:43--  https://caltech.box.com/public/static/0g7lnuieg8jxlxswrssdtz809gus75ek.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/0g7lnuieg8jxlxswrssdtz809gus75ek.gz [following]
    --2020-01-16 18:56:43--  https://caltech.app.box.com/public/static/0g7lnuieg8jxlxswrssdtz809gus75ek.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!79kAcHiZNW1K0g5DJlFaxIrBmP9Lq0jf5EpmyiR7Sgmpy5ctAFebeM-PuXcpFbOoZEVu0hOonAmRI11mSlDHIhd1ztkuqmhijupOwUo7xeD_Jl68Q_E_Vs9Jz_MU6SVcuHDI5OLiKU6pniutDYlW8FTIVxkdNG2nw9C99R1P-nbP4kj6KvqGcipyBI-4eA4OnXMZde9VFAYCGYl2RGZSph0rQElMBLu0urlryCZ2PCuX0uilIfwLVcNnXCZYI_fYR1__Mrlw9lvqjSovNgNZQzjuOT07UTailKUdPgOYlpDBqqmJGlOkZ2nunT5LMEZdiMl5U5t2mvYbnOwDiM05AGuANMoOBzkW4ysBJMPuKFV8M6YIUfQbZ6Pjh8RO3TKzCdAO_MTR__q1LElgeVWWD3hE_DFc5nJ19VsLfbdXwJca9ZFIsRsFF4yQRhYZIyi5IeI-pp0rBwQ-Ze8LQZBbH2U1_A6hkPwsE5Lrqlwc7wT2YFalTtNA6fbQc-6qzeQ70iiTEZiq11jJCfiHKMqijXzX8ctXu3aWg9XnrEhOsAzrejMVH6NrcbS_agMkEp5zJF1Bv3yqRhta6iAlLI7OE6-Ccobtjpqgi2zwK3CppNPR65ORDdNSRtPqASkyAjzdbImfPioH71zKoOGXUEq7IcyDlN-Sc2B799voZko_pjbBVwkaJraT1yYaHyuHhHfFF-U-O_T0OQkPnbikjUAZJsazPd-hh8h_Syv1LyAKNMLPAMEfD3wfJuqoEpGmDfEX_h1uaBQOZk8NGlrXfJ0LJl3uMsb60OKA4_Q7NNyKK-B0Pc0A41zzpSURCs37vvG6Dm_uZY5xCCKwL85fBvUuKycZS7X5icUmbGb8Szmpysltr4UpZtFmJ0-4WDAT0a9d0vNjEKsaYQeMSnQpocmLd52Q_6HJlP7aB_GugleSsn8bOTbwax6UAdt816rXQuUm30Jx1ZsjuErfb3flmf7jZ1O1gVzsgue0-wzFdRRgabhWs_kxajgmEn_UnhzrsNFTkfgu06LTvaX-z4N3VM4JotiC-wsMJLusgZOaMhgou_z9zjR7XUx_1u3AsPm8gUUUeH5mx-WVxRcvqLSd7iLOqy3thk-DWb0700eAyocZj-V5b2Tu9OSfeT7x9lPv9bsipB9z8Gvtdq8-_jgmc80Fq0s5NfGthlD7M5eJ_jVfXaj-dqwGTC6jvo3KpDggzsWVYvyZgEkL9my3yXKUqBAaEIHhMAjgVbkNGc3J_iaK7UiJvQgjNy2hcVWnIH1mcd_o3gVYSkwI8FJcmgPCEx3v_x9HrIu0Z5qSmnCn0q-ZbmuqlA443AOCv3sG99AH7CJxus25S_Fku_pzeIUCCCXbQmQBmeMEeM503xzfNo3lj7U0Oi4yaXTZoUMIwWwu_UJNJKU76gb9YbjzVcS-5-sc-lGaShA./download [following]
    --2020-01-16 18:56:44--  https://public.boxcloud.com/d/1/b1!79kAcHiZNW1K0g5DJlFaxIrBmP9Lq0jf5EpmyiR7Sgmpy5ctAFebeM-PuXcpFbOoZEVu0hOonAmRI11mSlDHIhd1ztkuqmhijupOwUo7xeD_Jl68Q_E_Vs9Jz_MU6SVcuHDI5OLiKU6pniutDYlW8FTIVxkdNG2nw9C99R1P-nbP4kj6KvqGcipyBI-4eA4OnXMZde9VFAYCGYl2RGZSph0rQElMBLu0urlryCZ2PCuX0uilIfwLVcNnXCZYI_fYR1__Mrlw9lvqjSovNgNZQzjuOT07UTailKUdPgOYlpDBqqmJGlOkZ2nunT5LMEZdiMl5U5t2mvYbnOwDiM05AGuANMoOBzkW4ysBJMPuKFV8M6YIUfQbZ6Pjh8RO3TKzCdAO_MTR__q1LElgeVWWD3hE_DFc5nJ19VsLfbdXwJca9ZFIsRsFF4yQRhYZIyi5IeI-pp0rBwQ-Ze8LQZBbH2U1_A6hkPwsE5Lrqlwc7wT2YFalTtNA6fbQc-6qzeQ70iiTEZiq11jJCfiHKMqijXzX8ctXu3aWg9XnrEhOsAzrejMVH6NrcbS_agMkEp5zJF1Bv3yqRhta6iAlLI7OE6-Ccobtjpqgi2zwK3CppNPR65ORDdNSRtPqASkyAjzdbImfPioH71zKoOGXUEq7IcyDlN-Sc2B799voZko_pjbBVwkaJraT1yYaHyuHhHfFF-U-O_T0OQkPnbikjUAZJsazPd-hh8h_Syv1LyAKNMLPAMEfD3wfJuqoEpGmDfEX_h1uaBQOZk8NGlrXfJ0LJl3uMsb60OKA4_Q7NNyKK-B0Pc0A41zzpSURCs37vvG6Dm_uZY5xCCKwL85fBvUuKycZS7X5icUmbGb8Szmpysltr4UpZtFmJ0-4WDAT0a9d0vNjEKsaYQeMSnQpocmLd52Q_6HJlP7aB_GugleSsn8bOTbwax6UAdt816rXQuUm30Jx1ZsjuErfb3flmf7jZ1O1gVzsgue0-wzFdRRgabhWs_kxajgmEn_UnhzrsNFTkfgu06LTvaX-z4N3VM4JotiC-wsMJLusgZOaMhgou_z9zjR7XUx_1u3AsPm8gUUUeH5mx-WVxRcvqLSd7iLOqy3thk-DWb0700eAyocZj-V5b2Tu9OSfeT7x9lPv9bsipB9z8Gvtdq8-_jgmc80Fq0s5NfGthlD7M5eJ_jVfXaj-dqwGTC6jvo3KpDggzsWVYvyZgEkL9my3yXKUqBAaEIHhMAjgVbkNGc3J_iaK7UiJvQgjNy2hcVWnIH1mcd_o3gVYSkwI8FJcmgPCEx3v_x9HrIu0Z5qSmnCn0q-ZbmuqlA443AOCv3sG99AH7CJxus25S_Fku_pzeIUCCCXbQmQBmeMEeM503xzfNo3lj7U0Oi4yaXTZoUMIwWwu_UJNJKU76gb9YbjzVcS-5-sc-lGaShA./download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 973374355 (928M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L002_R2_001.fastq.gz’
    
    1k_hgmm_v3_S1_L002_ 100%[===================>] 928.28M  13.0MB/s    in 75s     
    
    2020-01-16 18:57:59 (12.3 MB/s) - ‘1k_hgmm_v3_S1_L002_R2_001.fastq.gz’ saved [973374355/973374355]
    
    --2020-01-16 18:58:00--  https://caltech.box.com/shared/static/0avmybuxqcw8haa1hf0n72oyb8zriiuu.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/0avmybuxqcw8haa1hf0n72oyb8zriiuu.gz [following]
    --2020-01-16 18:58:01--  https://caltech.box.com/public/static/0avmybuxqcw8haa1hf0n72oyb8zriiuu.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/0avmybuxqcw8haa1hf0n72oyb8zriiuu.gz [following]
    --2020-01-16 18:58:01--  https://caltech.app.box.com/public/static/0avmybuxqcw8haa1hf0n72oyb8zriiuu.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!zHrLG8OVxmbu_X5CCalwcI9XsSk7skfol3wOewRa56Hh9VPV0leflqfgU2rRSijvIjZ4Ro-jMVv0IWuh4koUegkVcoA7BnwUObdAITHF7CoIROzymgqEFYNucz19Xj400z6VJREgqtuADoCurcPaDjgYNeFtpuw6sneQpa0GqTi256k1hVbcxsbU_tmbBygR9TE1bHjNsrAqpu_J-2u_A9ppq-JHToyKwktrckM7bXshBgtIYMScGTYPgqaMnFQixuzhEb8e0jzdL7GFC84nBTkIvgaS7pExnd10ER3cYqOPpZrASu97cl0dZ6wxIvqYl85-VFchjZXUwxc5afybt_lKOqSp8iMoPgggnFbZlFz261oQ3NvRAwvm5oFWWyZNAggGVorUweh-UQARGRYU4JD9VZKElwKD758BmYwa3pmGNhf5W95hSLze5Tpp47n8Hdl40QpPN0V60_y2nz2f7x-eMtOuoxU_KoWjMGG9Nr6WHKgFXpv-kKq9rAeTsmepkMFQDQmkKsvtQzi-9eNEg1DaBCBgLkdBz9a-1LhBut4Jx8D54-uiroZCG5bvBLryJlYuNtkjAW8ickBx9_KBcDNBhQsN0RyeN3DIhTlHTo6EC41HkInwoYSMw0Nmq3zU-jtU4wRxz7b49w1C7foZ0miiONEz1azwAb9eN7PBQatyPfNbPAqaZo2z09fwjthYFrv0gSEYBkweo-Wn6HCUVeUdpnYd3j4F1KEA8KWmMPw7ZG-e5IO9jvChfUBtKXmFTDkECC-f_ORBlG1q6WuP6hgFZpuI1vWnKwVqHrx7CKHaazb7oNKz0M73F8Sbd5uc63v4i0I3crPsBBbrX2GwKqWiSm23TkZrBxOsIAkKtV1Vfjic4ji4UJ4W7_iP5FxyVrMLBE8rJpng53sochm0k4zx5ZcPvJPUz765el3U4aV_D4sf-hA66yEy_W9EG7zW4W2cZY1k9jrNCavoUP9z8zS9d-uNDeDlbBKfd72TYDers2xmoqSNWD447fiSGQ_YjKAQqhUTxPstLbdg3b2zzgm3hFccjMaR2bgmJxz8XvBGKGVnMochQKC89Asy0jnWAdjfA9gjidBe20oyTHoWLiNUuDl-L3j_cR4Aa-dc4ipXAQj31IciinSx3M54DvkqHVhYHDlzuagrfFqGzEF7grhJXs_sQHmFkUQsaTdF6eDZNVt_8L8w3Raw6nwQXT4Y3gG6XHIGwBSJeNTMMMEZdHZFXB5gahsZkFKJy_3sq2cPC8PgM3du5E4qv8ZPzP98AJfdY81khyCQefsshOw7hcNMqS_zL8r12AIlH_WR524K89BvHwDq6y7D7anHez39wJTmZYZOnYQ_-OSyToo_6blIztoIuKpqLL3u4NwaO5jEV5GqhExBVsrw4U_dYtJaKbyK-OCu4HW2G_IGxOBVZ9SqBe0./download [following]
    --2020-01-16 18:58:02--  https://public.boxcloud.com/d/1/b1!zHrLG8OVxmbu_X5CCalwcI9XsSk7skfol3wOewRa56Hh9VPV0leflqfgU2rRSijvIjZ4Ro-jMVv0IWuh4koUegkVcoA7BnwUObdAITHF7CoIROzymgqEFYNucz19Xj400z6VJREgqtuADoCurcPaDjgYNeFtpuw6sneQpa0GqTi256k1hVbcxsbU_tmbBygR9TE1bHjNsrAqpu_J-2u_A9ppq-JHToyKwktrckM7bXshBgtIYMScGTYPgqaMnFQixuzhEb8e0jzdL7GFC84nBTkIvgaS7pExnd10ER3cYqOPpZrASu97cl0dZ6wxIvqYl85-VFchjZXUwxc5afybt_lKOqSp8iMoPgggnFbZlFz261oQ3NvRAwvm5oFWWyZNAggGVorUweh-UQARGRYU4JD9VZKElwKD758BmYwa3pmGNhf5W95hSLze5Tpp47n8Hdl40QpPN0V60_y2nz2f7x-eMtOuoxU_KoWjMGG9Nr6WHKgFXpv-kKq9rAeTsmepkMFQDQmkKsvtQzi-9eNEg1DaBCBgLkdBz9a-1LhBut4Jx8D54-uiroZCG5bvBLryJlYuNtkjAW8ickBx9_KBcDNBhQsN0RyeN3DIhTlHTo6EC41HkInwoYSMw0Nmq3zU-jtU4wRxz7b49w1C7foZ0miiONEz1azwAb9eN7PBQatyPfNbPAqaZo2z09fwjthYFrv0gSEYBkweo-Wn6HCUVeUdpnYd3j4F1KEA8KWmMPw7ZG-e5IO9jvChfUBtKXmFTDkECC-f_ORBlG1q6WuP6hgFZpuI1vWnKwVqHrx7CKHaazb7oNKz0M73F8Sbd5uc63v4i0I3crPsBBbrX2GwKqWiSm23TkZrBxOsIAkKtV1Vfjic4ji4UJ4W7_iP5FxyVrMLBE8rJpng53sochm0k4zx5ZcPvJPUz765el3U4aV_D4sf-hA66yEy_W9EG7zW4W2cZY1k9jrNCavoUP9z8zS9d-uNDeDlbBKfd72TYDers2xmoqSNWD447fiSGQ_YjKAQqhUTxPstLbdg3b2zzgm3hFccjMaR2bgmJxz8XvBGKGVnMochQKC89Asy0jnWAdjfA9gjidBe20oyTHoWLiNUuDl-L3j_cR4Aa-dc4ipXAQj31IciinSx3M54DvkqHVhYHDlzuagrfFqGzEF7grhJXs_sQHmFkUQsaTdF6eDZNVt_8L8w3Raw6nwQXT4Y3gG6XHIGwBSJeNTMMMEZdHZFXB5gahsZkFKJy_3sq2cPC8PgM3du5E4qv8ZPzP98AJfdY81khyCQefsshOw7hcNMqS_zL8r12AIlH_WR524K89BvHwDq6y7D7anHez39wJTmZYZOnYQ_-OSyToo_6blIztoIuKpqLL3u4NwaO5jEV5GqhExBVsrw4U_dYtJaKbyK-OCu4HW2G_IGxOBVZ9SqBe0./download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 421486566 (402M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L003_R1_001.fastq.gz’
    
    1k_hgmm_v3_S1_L003_ 100%[===================>] 401.96M  16.8MB/s    in 25s     
    
    2020-01-16 18:58:28 (16.2 MB/s) - ‘1k_hgmm_v3_S1_L003_R1_001.fastq.gz’ saved [421486566/421486566]
    
    --2020-01-16 18:58:29--  https://caltech.box.com/shared/static/hp10z2yr8u3lbzoj1qflz83r2v9ohs6q.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/hp10z2yr8u3lbzoj1qflz83r2v9ohs6q.gz [following]
    --2020-01-16 18:58:29--  https://caltech.box.com/public/static/hp10z2yr8u3lbzoj1qflz83r2v9ohs6q.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/hp10z2yr8u3lbzoj1qflz83r2v9ohs6q.gz [following]
    --2020-01-16 18:58:29--  https://caltech.app.box.com/public/static/hp10z2yr8u3lbzoj1qflz83r2v9ohs6q.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!LNaWTLUQ-vxEPA9_tHQ4uDYdxzj-1zQ5BQ6bOusms_Y99Z8R8PZNzG_fYT4TROauq-YdTB9GtbP4dfwW_zRVQJ8CGgLSskQSK0I06Z_JI9uY0lvn39jSklO1R9c0EJk8uYJGyPErMQEIv2PGFv0R6X-loY3xReq_f8qlO6nH3xycTSaVs3tJ5DNQQvPyGa0UVXs2zpKVI_jsxCvOI_rn-_W-AS5hurrs4NSi_KQfe-phIxvzJoeN35otkm7WFj4Uqr9yY9JR6rdar458hlWkM35o-hBdoEqfEX7bjbUwVYhCLUX-KVdym_gNuWpLlUokqiSB7mRbila0_zyiOnuYp41r1uo4B_MQfGefTPi4Sft-uDSEMbpebxDvwgVO-6TXrjy41ltLydUSTl8npGK0HpGiwfq-4dyZGf9HC6AlVcsnW3i6o-naZaP0Q25Vy2YGVfXPODqEo8HF0VRYUC6JuisLHQbPoN_2MCD3Ba-rY60XzosXWzTojVRQWbBRSUZR4Bj-fYfo7TPndzkmbVUgnmYpi_twtu0O8B9VwkKYBR5EYrVsPiR00O1TxrXMpApFnyNCPG5PlNmktvYX_yst2dSsZJtxLoykEJIKTCHOVfpze_J8YJhBcGIFmlLp7I7I8dlxJlZ125zWRWlIEg0gv3ihFtl7Xm6F_0KlgZ67hsjXyW6Wi45hiyOxOsO1n8z9n0BlMMPpOHX_DsC3kz0D4SM-9XKHivQ8fWcMIND5ZWY0azA0kA0Hk0r22DgKgBozCl2QTw9_2taOyIMeZaRMCxVDX6Eh2X4r6XAMSjzmzpVlQXQwm1u4-dBMokk8ExwLmi1ViKiCfoOpmE6SSkE2tODHCGTwkEkHWbUe1zz2cEhTsjxwv1P2Frq8RrcAn4cn_3bZ5EHhzE9Rsrp5xQFAd60sAo3s76fvholiwvj4v16bT8FSxBuL456yOC5PmyKAkanksb8UAyZiMM2RwzRmVLr4WGeRehVPwl4E11LkIiKCm3OVZ2aYx3lKMNdAw9VqvQn3rkT1fsS86iz7UGnHS-HIWi6RtMJRFs4zFcsHmUozCUNC9gbkqCWPdpYUffYjzvD6Mrw7Lj2lTjfKlVCF1-YnbWNxREmSyREwCt-MbZ31C08p0XFCsPUhUQTfSHLzmghuCB2MHhDJhN_hIftJGflZUq-4pQksPHR3uXl2P-zZiObYWDzVt5KqmKUbVn0FVmVYruayzYYJDZ9NQF_jSJ-RNgPXT6kVbZqJfu5_lME07vaKluqNHhgrh3m5TAb7Acy6g7kaIGa-Ap4vQ1DYDooIHPbDrQGSbaiEW_p3KqNG4eFjgmoHarBFAcN3CqcJB2gmykPa8nCLn9oWt4Va6Ag0YXg9F8R9Va37PGTe9-HIzi0MVn-RAAmz-1M5WFnonQ0KUfTB5qfGpumhQ8MB3ELERMg./download [following]
    --2020-01-16 18:58:30--  https://public.boxcloud.com/d/1/b1!LNaWTLUQ-vxEPA9_tHQ4uDYdxzj-1zQ5BQ6bOusms_Y99Z8R8PZNzG_fYT4TROauq-YdTB9GtbP4dfwW_zRVQJ8CGgLSskQSK0I06Z_JI9uY0lvn39jSklO1R9c0EJk8uYJGyPErMQEIv2PGFv0R6X-loY3xReq_f8qlO6nH3xycTSaVs3tJ5DNQQvPyGa0UVXs2zpKVI_jsxCvOI_rn-_W-AS5hurrs4NSi_KQfe-phIxvzJoeN35otkm7WFj4Uqr9yY9JR6rdar458hlWkM35o-hBdoEqfEX7bjbUwVYhCLUX-KVdym_gNuWpLlUokqiSB7mRbila0_zyiOnuYp41r1uo4B_MQfGefTPi4Sft-uDSEMbpebxDvwgVO-6TXrjy41ltLydUSTl8npGK0HpGiwfq-4dyZGf9HC6AlVcsnW3i6o-naZaP0Q25Vy2YGVfXPODqEo8HF0VRYUC6JuisLHQbPoN_2MCD3Ba-rY60XzosXWzTojVRQWbBRSUZR4Bj-fYfo7TPndzkmbVUgnmYpi_twtu0O8B9VwkKYBR5EYrVsPiR00O1TxrXMpApFnyNCPG5PlNmktvYX_yst2dSsZJtxLoykEJIKTCHOVfpze_J8YJhBcGIFmlLp7I7I8dlxJlZ125zWRWlIEg0gv3ihFtl7Xm6F_0KlgZ67hsjXyW6Wi45hiyOxOsO1n8z9n0BlMMPpOHX_DsC3kz0D4SM-9XKHivQ8fWcMIND5ZWY0azA0kA0Hk0r22DgKgBozCl2QTw9_2taOyIMeZaRMCxVDX6Eh2X4r6XAMSjzmzpVlQXQwm1u4-dBMokk8ExwLmi1ViKiCfoOpmE6SSkE2tODHCGTwkEkHWbUe1zz2cEhTsjxwv1P2Frq8RrcAn4cn_3bZ5EHhzE9Rsrp5xQFAd60sAo3s76fvholiwvj4v16bT8FSxBuL456yOC5PmyKAkanksb8UAyZiMM2RwzRmVLr4WGeRehVPwl4E11LkIiKCm3OVZ2aYx3lKMNdAw9VqvQn3rkT1fsS86iz7UGnHS-HIWi6RtMJRFs4zFcsHmUozCUNC9gbkqCWPdpYUffYjzvD6Mrw7Lj2lTjfKlVCF1-YnbWNxREmSyREwCt-MbZ31C08p0XFCsPUhUQTfSHLzmghuCB2MHhDJhN_hIftJGflZUq-4pQksPHR3uXl2P-zZiObYWDzVt5KqmKUbVn0FVmVYruayzYYJDZ9NQF_jSJ-RNgPXT6kVbZqJfu5_lME07vaKluqNHhgrh3m5TAb7Acy6g7kaIGa-Ap4vQ1DYDooIHPbDrQGSbaiEW_p3KqNG4eFjgmoHarBFAcN3CqcJB2gmykPa8nCLn9oWt4Va6Ag0YXg9F8R9Va37PGTe9-HIzi0MVn-RAAmz-1M5WFnonQ0KUfTB5qfGpumhQ8MB3ELERMg./download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 970486795 (926M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L003_R2_001.fastq.gz’
    
    1k_hgmm_v3_S1_L003_ 100%[===================>] 925.53M  15.6MB/s    in 60s     
    
    2020-01-16 18:59:30 (15.5 MB/s) - ‘1k_hgmm_v3_S1_L003_R2_001.fastq.gz’ saved [970486795/970486795]
    
    --2020-01-16 18:59:32--  https://caltech.box.com/shared/static/fx8fduedje53dvf3xixyyaqzugn7yy85.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/fx8fduedje53dvf3xixyyaqzugn7yy85.gz [following]
    --2020-01-16 18:59:32--  https://caltech.box.com/public/static/fx8fduedje53dvf3xixyyaqzugn7yy85.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/fx8fduedje53dvf3xixyyaqzugn7yy85.gz [following]
    --2020-01-16 18:59:33--  https://caltech.app.box.com/public/static/fx8fduedje53dvf3xixyyaqzugn7yy85.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!imbmg2pYPv4x_SlBgRNQE5NOR5aNqtkjYk-YINStXZGTaCj8buOftPVEncitP7br6ug5kpmYANYKROeXMtjOfORToRAZg24MIIVLjG7Z7tuYdgBMo-2o2S8YCrolfWwUMYgsv0y9qSb0RNQZx6OpB1Tbq9eiUjn9YW7vKQ1_QA6tY10HgzSI_HbcPPStqpen8U0JasrqZ8g3OUw7MRx2m5JYuDYV3BbCiMVdah5HDBk_Ce53T5cJL8uKuMoxNHqgohQ6vL8pLZLlLCqoPjbhKWeTyxnyVFTxL-ke_OyllVx8kaC3F4HqNwGf1fWm6bcxi8H7-0mnlBY36qP5MrZ3sw6EPdP6Wq3o-4WcMggxXn0cxqxRkXbWfZm1l0tXdY_MtzavlsLZgx-biwOvAhZ6vFwL0hTAxuRdX59nxersZ3bgDvW2HJZnHTlShvRJzEIjZeJzp1lFQPz0MlQYiNh2Tj5T182PED8BoFWvrSd2Ft0nmZ3dkOTvqYikpI7kxR0aR6TCtxZQZXYssb5QYmvJ-wfhBY7ta-Kxezx9rxQvBZWGuv1tUkULoGt9hlB_MK1knySnTuYaZSHtpa46vzuLKQUVsy6rdfHsjjhkPfSP8vsz6OsunPIesENH7OaIPOAPcaOSdAaSKL7TNUeGa0_E5RcUugz8adLWcER12bG6zeo8-wqbYIr32IuH1VRIW6U2szmSMdJ60sssRXHeapkuPcyOlnq4kqrtbXYkATHsJDw0PAd88_B1Bl16LEEFy6BpgcL1gLBCYDOZivVEzv2ZJh2o5PptBIPnWWbdCZxIfvMcFCUo1Z9OA-BYlUL9YrsaAZP3Ndb_U78Vv6y0H0ENJudx8LchOXWrV6hnHuILUpq16KmyxRWTF4CPey05hNLhSQGvv9KSIR1E05sU7lQHjFK3BttPkoQlk1oA8aAjXASIG72lHNztLblGj9jxrOBJ6k0sfSfC-aRgT0MntN_YghdiZq8jmRqQP0jx34D2gAagtPhyC7JhmAA44fEpx2Syjr2yS4mH3IdWCfPAKeG4M3MvpLKmTXi9enaYqHXCPrqoll_lYB8VwKXDSdzIZayUyCCBLGd4rLuVGCuceeQ8xJxoRG4LDAqwf97-Qh5tMucrZ3K6aRGBT-L0YqZROgcE_Wtvw6oRFt7F-6y7a3eVx2BoKagRPm11GEOgKFZfZi8IOL5seAs9PSqTqwdZN9_FmcB0M5FQDTyAfHy-Xb0TsUmmz_UceeMxz5QQcqY-F36YoCw3UUwv4QbgTTahuRcpHFCfOh1mK3k2glKu_2LPxhIsDBhuijDaMwO-o5HHnsuYmBC1VhZfmWE9suqaC4RZxGCyHrYLhuGytTao99b_A1Mov-eK0Zg7SohrntkpvncgUa1kbEOGE3yP9eOvT985RAa2uxkGonWxQpxOeiq1awWwT79Q/download [following]
    --2020-01-16 18:59:33--  https://public.boxcloud.com/d/1/b1!imbmg2pYPv4x_SlBgRNQE5NOR5aNqtkjYk-YINStXZGTaCj8buOftPVEncitP7br6ug5kpmYANYKROeXMtjOfORToRAZg24MIIVLjG7Z7tuYdgBMo-2o2S8YCrolfWwUMYgsv0y9qSb0RNQZx6OpB1Tbq9eiUjn9YW7vKQ1_QA6tY10HgzSI_HbcPPStqpen8U0JasrqZ8g3OUw7MRx2m5JYuDYV3BbCiMVdah5HDBk_Ce53T5cJL8uKuMoxNHqgohQ6vL8pLZLlLCqoPjbhKWeTyxnyVFTxL-ke_OyllVx8kaC3F4HqNwGf1fWm6bcxi8H7-0mnlBY36qP5MrZ3sw6EPdP6Wq3o-4WcMggxXn0cxqxRkXbWfZm1l0tXdY_MtzavlsLZgx-biwOvAhZ6vFwL0hTAxuRdX59nxersZ3bgDvW2HJZnHTlShvRJzEIjZeJzp1lFQPz0MlQYiNh2Tj5T182PED8BoFWvrSd2Ft0nmZ3dkOTvqYikpI7kxR0aR6TCtxZQZXYssb5QYmvJ-wfhBY7ta-Kxezx9rxQvBZWGuv1tUkULoGt9hlB_MK1knySnTuYaZSHtpa46vzuLKQUVsy6rdfHsjjhkPfSP8vsz6OsunPIesENH7OaIPOAPcaOSdAaSKL7TNUeGa0_E5RcUugz8adLWcER12bG6zeo8-wqbYIr32IuH1VRIW6U2szmSMdJ60sssRXHeapkuPcyOlnq4kqrtbXYkATHsJDw0PAd88_B1Bl16LEEFy6BpgcL1gLBCYDOZivVEzv2ZJh2o5PptBIPnWWbdCZxIfvMcFCUo1Z9OA-BYlUL9YrsaAZP3Ndb_U78Vv6y0H0ENJudx8LchOXWrV6hnHuILUpq16KmyxRWTF4CPey05hNLhSQGvv9KSIR1E05sU7lQHjFK3BttPkoQlk1oA8aAjXASIG72lHNztLblGj9jxrOBJ6k0sfSfC-aRgT0MntN_YghdiZq8jmRqQP0jx34D2gAagtPhyC7JhmAA44fEpx2Syjr2yS4mH3IdWCfPAKeG4M3MvpLKmTXi9enaYqHXCPrqoll_lYB8VwKXDSdzIZayUyCCBLGd4rLuVGCuceeQ8xJxoRG4LDAqwf97-Qh5tMucrZ3K6aRGBT-L0YqZROgcE_Wtvw6oRFt7F-6y7a3eVx2BoKagRPm11GEOgKFZfZi8IOL5seAs9PSqTqwdZN9_FmcB0M5FQDTyAfHy-Xb0TsUmmz_UceeMxz5QQcqY-F36YoCw3UUwv4QbgTTahuRcpHFCfOh1mK3k2glKu_2LPxhIsDBhuijDaMwO-o5HHnsuYmBC1VhZfmWE9suqaC4RZxGCyHrYLhuGytTao99b_A1Mov-eK0Zg7SohrntkpvncgUa1kbEOGE3yP9eOvT985RAa2uxkGonWxQpxOeiq1awWwT79Q/download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 428614106 (409M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L004_R1_001.fastq.gz’
    
    1k_hgmm_v3_S1_L004_ 100%[===================>] 408.76M  14.8MB/s    in 27s     
    
    2020-01-16 19:00:01 (14.9 MB/s) - ‘1k_hgmm_v3_S1_L004_R1_001.fastq.gz’ saved [428614106/428614106]
    
    --2020-01-16 19:00:03--  https://caltech.box.com/shared/static/lpt6uzmueh1l2vx71nvsdj3pwqh8z3ak.gz
    Resolving caltech.box.com (caltech.box.com)... 103.116.4.197
    Connecting to caltech.box.com (caltech.box.com)|103.116.4.197|:443... connected.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: /public/static/lpt6uzmueh1l2vx71nvsdj3pwqh8z3ak.gz [following]
    --2020-01-16 19:00:03--  https://caltech.box.com/public/static/lpt6uzmueh1l2vx71nvsdj3pwqh8z3ak.gz
    Reusing existing connection to caltech.box.com:443.
    HTTP request sent, awaiting response... 301 Moved Permanently
    Location: https://caltech.app.box.com/public/static/lpt6uzmueh1l2vx71nvsdj3pwqh8z3ak.gz [following]
    --2020-01-16 19:00:03--  https://caltech.app.box.com/public/static/lpt6uzmueh1l2vx71nvsdj3pwqh8z3ak.gz
    Resolving caltech.app.box.com (caltech.app.box.com)... 103.116.4.199
    Connecting to caltech.app.box.com (caltech.app.box.com)|103.116.4.199|:443... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: https://public.boxcloud.com/d/1/b1!aR-u6N2LrOdq-EuVbJbBEUk_6DUe630nsy8NEYSVJo78KVIcz8zdjQ8iMc0ipefvzQ2XAGOevcum9kZ8qqSKR0xSqs420wqrhoyfIQO7NVqI5a-bHk8u-uS5RLcAByJddbj-XB1Iv2LM45DL0OKC1aW8UOrk-o1MqethJAtF0iJSccRI0GZBJstB8IXrDuhhF0pYd8hh1DOQVasV1FL6-2VpFNA7DHlqnh2Ox80X0ratcJAncbtChZhnU48xVcKDmAOehay9a8eOhjck7ej94n1EK7AOx7MI7dvSUA8yUfkjHK4jCWVC6jlexRNymoaa4P_zDl1wGJO_u581-dhKxXE0ht4zwNuSo8EHEaYAI8q_XAHxTQzMFvu6Rp6uTUc7jP3y_9f9GiiwtBU496QgBbTaZ4CcifmXJa_qb_eIR1iUHmjpnVgSS1XK-8Zfz37Y7DUFPKJz4daBCozgU92F2Oe6_TplzA4exVx-QzVDMD-btpWAOOibAQ1lfqsEXjEPieOWUs9kVlNUfG4kmK34YqYMAkRnCsMzLoy0GtGqyLOPSMGwfk170LeNjJn-nwGs_XBL4gttrqjEcnppHHaEcC-8Hy8wQQ6BBrjiDQ5X-ZfytkXny_jZ7IFOPIOkg3j8ykIfOuoM8gTaKRSuuxGPRUTBJLxSH9noHEGoIALqWgZ18YdzVzGeNnm6ANMezOOivt7CLtdJ2Ctg3Qa-C0qdZj_fnbab3JO8dOEdiTlgTob2sA3_3QNh3Hnp7dtpHiEyAXYoarOnuk0ylDDRDWl9yxuuHZXFSc2FkQywkMWz0lbNLAoiVL8v5HfDXA6_DAOGgjFnv7YEOGA5I3mOBUhb7eGjJMDSfk-eWkf3m_ISe2fPM002vLFU2BuXBxItcUXpvpGXYFq_Eio-Fj7a4N6zjfOh5VhO__oa6LO0H7ZBkvkPZv4Z77ON4r4j50HqcwAaDP19gpSm4ac8qXV3aQAKdq2b3Cp5-by-4iL4iNFu3XEnYw562uBgKmUQtnBRIr9JXcH5_73c3WP7WFFRfiddmpIBWUOIOLXNZLZ5ge2Is8t6wXGiyhgPNwDjvoJk18evgoU5fnjQDR4x_XHbKr2rpA6XGMguWqOHA9F1I8iCW0UV1tupLzs6fslujlnsfmci-ih33YBdzwgeDkzvT8zCkxdXxktPWES1a3X06NUp7oOa1FaC8AOfBBcaA7JozlM2om369I5WTqe-UCzd5Pg_LODf2hVFxtsNipjoHMESLcJwWJTUnE5o5ttsOYnhy-GIFbeqZsd7rhVIKrBhPx2oSavUMsJtYM44cp3BEdbSKXKj5pj8nQW9wCLVPgz5AyoyMJHZ72He8ax3rZFxk15uamgy9ywrc4q9DukMpuLmVGN9lG9eDm2IXIAO_QeBmPRqQQ07H7CJFARP0SgCTG56sg24GyE./download [following]
    --2020-01-16 19:00:04--  https://public.boxcloud.com/d/1/b1!aR-u6N2LrOdq-EuVbJbBEUk_6DUe630nsy8NEYSVJo78KVIcz8zdjQ8iMc0ipefvzQ2XAGOevcum9kZ8qqSKR0xSqs420wqrhoyfIQO7NVqI5a-bHk8u-uS5RLcAByJddbj-XB1Iv2LM45DL0OKC1aW8UOrk-o1MqethJAtF0iJSccRI0GZBJstB8IXrDuhhF0pYd8hh1DOQVasV1FL6-2VpFNA7DHlqnh2Ox80X0ratcJAncbtChZhnU48xVcKDmAOehay9a8eOhjck7ej94n1EK7AOx7MI7dvSUA8yUfkjHK4jCWVC6jlexRNymoaa4P_zDl1wGJO_u581-dhKxXE0ht4zwNuSo8EHEaYAI8q_XAHxTQzMFvu6Rp6uTUc7jP3y_9f9GiiwtBU496QgBbTaZ4CcifmXJa_qb_eIR1iUHmjpnVgSS1XK-8Zfz37Y7DUFPKJz4daBCozgU92F2Oe6_TplzA4exVx-QzVDMD-btpWAOOibAQ1lfqsEXjEPieOWUs9kVlNUfG4kmK34YqYMAkRnCsMzLoy0GtGqyLOPSMGwfk170LeNjJn-nwGs_XBL4gttrqjEcnppHHaEcC-8Hy8wQQ6BBrjiDQ5X-ZfytkXny_jZ7IFOPIOkg3j8ykIfOuoM8gTaKRSuuxGPRUTBJLxSH9noHEGoIALqWgZ18YdzVzGeNnm6ANMezOOivt7CLtdJ2Ctg3Qa-C0qdZj_fnbab3JO8dOEdiTlgTob2sA3_3QNh3Hnp7dtpHiEyAXYoarOnuk0ylDDRDWl9yxuuHZXFSc2FkQywkMWz0lbNLAoiVL8v5HfDXA6_DAOGgjFnv7YEOGA5I3mOBUhb7eGjJMDSfk-eWkf3m_ISe2fPM002vLFU2BuXBxItcUXpvpGXYFq_Eio-Fj7a4N6zjfOh5VhO__oa6LO0H7ZBkvkPZv4Z77ON4r4j50HqcwAaDP19gpSm4ac8qXV3aQAKdq2b3Cp5-by-4iL4iNFu3XEnYw562uBgKmUQtnBRIr9JXcH5_73c3WP7WFFRfiddmpIBWUOIOLXNZLZ5ge2Is8t6wXGiyhgPNwDjvoJk18evgoU5fnjQDR4x_XHbKr2rpA6XGMguWqOHA9F1I8iCW0UV1tupLzs6fslujlnsfmci-ih33YBdzwgeDkzvT8zCkxdXxktPWES1a3X06NUp7oOa1FaC8AOfBBcaA7JozlM2om369I5WTqe-UCzd5Pg_LODf2hVFxtsNipjoHMESLcJwWJTUnE5o5ttsOYnhy-GIFbeqZsd7rhVIKrBhPx2oSavUMsJtYM44cp3BEdbSKXKj5pj8nQW9wCLVPgz5AyoyMJHZ72He8ax3rZFxk15uamgy9ywrc4q9DukMpuLmVGN9lG9eDm2IXIAO_QeBmPRqQQ07H7CJFARP0SgCTG56sg24GyE./download
    Resolving public.boxcloud.com (public.boxcloud.com)... 103.116.4.200
    Connecting to public.boxcloud.com (public.boxcloud.com)|103.116.4.200|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 984350311 (939M) [application/octet-stream]
    Saving to: ‘1k_hgmm_v3_S1_L004_R2_001.fastq.gz’
    
    1k_hgmm_v3_S1_L004_ 100%[===================>] 938.75M  14.8MB/s    in 66s     
    
    2020-01-16 19:01:11 (14.3 MB/s) - ‘1k_hgmm_v3_S1_L004_R2_001.fastq.gz’ saved [984350311/984350311]
    
    CPU times: user 4.07 s, sys: 923 ms, total: 4.99 s
    Wall time: 6min 36s


Then, we verify the integrity of the files we downloaded to make sure they were not corrupted during the download.


```
!md5sum -c checksums.txt --ignore-missing
```

    1k_hgmm_v3_S1_L001_R1_001.fastq.gz: OK
    1k_hgmm_v3_S1_L001_R2_001.fastq.gz: OK
    1k_hgmm_v3_S1_L002_R1_001.fastq.gz: OK
    1k_hgmm_v3_S1_L002_R2_001.fastq.gz: OK
    1k_hgmm_v3_S1_L003_R1_001.fastq.gz: OK
    1k_hgmm_v3_S1_L003_R2_001.fastq.gz: OK
    1k_hgmm_v3_S1_L004_R1_001.fastq.gz: OK
    1k_hgmm_v3_S1_L004_R2_001.fastq.gz: OK


### Install `kb`

Install `kb` for running the kallisto|bustools workflow.


```
!pip install git+https://github.com/pachterlab/kb_python@count-kite
```

    Collecting git+https://github.com/pachterlab/kb_python@count-kite
      Cloning https://github.com/pachterlab/kb_python (to revision count-kite) to /tmp/pip-req-build-a0qz7ipg
      Running command git clone -q https://github.com/pachterlab/kb_python /tmp/pip-req-build-a0qz7ipg
      Running command git checkout -b count-kite --track origin/count-kite
      Switched to a new branch 'count-kite'
      Branch 'count-kite' set up to track remote branch 'count-kite' from 'origin'.
    Collecting anndata>=0.6.22.post1
    [?25l  Downloading https://files.pythonhosted.org/packages/2b/72/87196c15f68d9865c31a43a10cf7c50bcbcedd5607d09f9aada0b3963103/anndata-0.6.22.post1-py3-none-any.whl (47kB)
    [K     |████████████████████████████████| 51kB 1.5MB/s 
    [?25hCollecting loompy>=3.0.6
    [?25l  Downloading https://files.pythonhosted.org/packages/36/52/74ed37ae5988522fbf87b856c67c4f80700e6452410b4cd80498c5f416f9/loompy-3.0.6.tar.gz (41kB)
    [K     |████████████████████████████████| 51kB 5.2MB/s 
    [?25hRequirement already satisfied: requests>=2.19.0 in /usr/local/lib/python3.6/dist-packages (from kb-python==0.24.4) (2.21.0)
    Collecting tqdm>=4.39.0
    [?25l  Downloading https://files.pythonhosted.org/packages/72/c9/7fc20feac72e79032a7c8138fd0d395dc6d8812b5b9edf53c3afd0b31017/tqdm-4.41.1-py2.py3-none-any.whl (56kB)
    [K     |████████████████████████████████| 61kB 6.0MB/s 
    [?25hRequirement already satisfied: h5py in /usr/local/lib/python3.6/dist-packages (from anndata>=0.6.22.post1->kb-python==0.24.4) (2.8.0)
    Requirement already satisfied: numpy~=1.14 in /usr/local/lib/python3.6/dist-packages (from anndata>=0.6.22.post1->kb-python==0.24.4) (1.17.5)
    Requirement already satisfied: natsort in /usr/local/lib/python3.6/dist-packages (from anndata>=0.6.22.post1->kb-python==0.24.4) (5.5.0)
    Requirement already satisfied: scipy~=1.0 in /usr/local/lib/python3.6/dist-packages (from anndata>=0.6.22.post1->kb-python==0.24.4) (1.4.1)
    Requirement already satisfied: pandas>=0.23.0 in /usr/local/lib/python3.6/dist-packages (from anndata>=0.6.22.post1->kb-python==0.24.4) (0.25.3)
    Requirement already satisfied: setuptools in /usr/local/lib/python3.6/dist-packages (from loompy>=3.0.6->kb-python==0.24.4) (42.0.2)
    Requirement already satisfied: numba in /usr/local/lib/python3.6/dist-packages (from loompy>=3.0.6->kb-python==0.24.4) (0.47.0)
    Requirement already satisfied: click in /usr/local/lib/python3.6/dist-packages (from loompy>=3.0.6->kb-python==0.24.4) (7.0)
    Collecting numpy-groupies
    [?25l  Downloading https://files.pythonhosted.org/packages/57/ae/18217b57ba3e4bb8a44ecbfc161ed065f6d1b90c75d404bd6ba8d6f024e2/numpy_groupies-0.9.10.tar.gz (43kB)
    [K     |████████████████████████████████| 51kB 5.0MB/s 
    [?25hRequirement already satisfied: urllib3<1.25,>=1.21.1 in /usr/local/lib/python3.6/dist-packages (from requests>=2.19.0->kb-python==0.24.4) (1.24.3)
    Requirement already satisfied: chardet<3.1.0,>=3.0.2 in /usr/local/lib/python3.6/dist-packages (from requests>=2.19.0->kb-python==0.24.4) (3.0.4)
    Requirement already satisfied: idna<2.9,>=2.5 in /usr/local/lib/python3.6/dist-packages (from requests>=2.19.0->kb-python==0.24.4) (2.8)
    Requirement already satisfied: certifi>=2017.4.17 in /usr/local/lib/python3.6/dist-packages (from requests>=2.19.0->kb-python==0.24.4) (2019.11.28)
    Requirement already satisfied: six in /usr/local/lib/python3.6/dist-packages (from h5py->anndata>=0.6.22.post1->kb-python==0.24.4) (1.12.0)
    Requirement already satisfied: python-dateutil>=2.6.1 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->anndata>=0.6.22.post1->kb-python==0.24.4) (2.6.1)
    Requirement already satisfied: pytz>=2017.2 in /usr/local/lib/python3.6/dist-packages (from pandas>=0.23.0->anndata>=0.6.22.post1->kb-python==0.24.4) (2018.9)
    Requirement already satisfied: llvmlite>=0.31.0dev0 in /usr/local/lib/python3.6/dist-packages (from numba->loompy>=3.0.6->kb-python==0.24.4) (0.31.0)
    Building wheels for collected packages: kb-python, loompy, numpy-groupies
      Building wheel for kb-python (setup.py) ... [?25l[?25hdone
      Created wheel for kb-python: filename=kb_python-0.24.4-cp36-none-any.whl size=80991434 sha256=b20b650c19860c1906c5f89f39c001fc874adf088a4ea686339d12b8d79bb949
      Stored in directory: /tmp/pip-ephem-wheel-cache-jjr0rnle/wheels/8e/56/56/c89223de74af26792675e82f4bb5223e7cf0d653a33038e34c
      Building wheel for loompy (setup.py) ... [?25l[?25hdone
      Created wheel for loompy: filename=loompy-3.0.6-cp36-none-any.whl size=47896 sha256=a5e692dd7ff61ebcf5e8a8c6fbbe7b21d7c62ef647f1cf05c685a47022b8ad3f
      Stored in directory: /root/.cache/pip/wheels/f9/a4/90/5a98ad83419732b0fba533b81a2a52ba3dbe230a936ca4cdc9
      Building wheel for numpy-groupies (setup.py) ... [?25l[?25hdone
      Created wheel for numpy-groupies: filename=numpy_groupies-0+unknown-cp36-none-any.whl size=28044 sha256=cad74d3e67d56a0982d83056af061c8f0c0f78550f14af52834f6523756b1c7d
      Stored in directory: /root/.cache/pip/wheels/30/ac/83/64d5f9293aeaec63f9539142fc629a41af064cae1b3d8d94aa
    Successfully built kb-python loompy numpy-groupies
    Installing collected packages: anndata, numpy-groupies, loompy, tqdm, kb-python
      Found existing installation: tqdm 4.28.1
        Uninstalling tqdm-4.28.1:
          Successfully uninstalled tqdm-4.28.1
    Successfully installed anndata-0.6.22.post1 kb-python-0.24.4 loompy-3.0.6 numpy-groupies-0+unknown tqdm-4.41.1




### Download human and mouse reference files

We will download the following files from Ensembl:
* Mouse genome (FASTA)
* Mouse genome annotations (GTF)
* Human genome (FASTA)
* Human genome annotations (GTF)


```
%%time
!wget ftp://ftp.ensembl.org/pub/release-98/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna.primary_assembly.fa.gz
!wget ftp://ftp.ensembl.org/pub/release-98/gtf/mus_musculus/Mus_musculus.GRCm38.98.gtf.gz
!wget ftp://ftp.ensembl.org/pub/release-98/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
!wget ftp://ftp.ensembl.org/pub/release-98/gtf/homo_sapiens/Homo_sapiens.GRCh38.98.gtf.gz
```

    --2020-01-16 19:02:01--  ftp://ftp.ensembl.org/pub/release-98/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna.primary_assembly.fa.gz
               => ‘Mus_musculus.GRCm38.dna.primary_assembly.fa.gz’
    Resolving ftp.ensembl.org (ftp.ensembl.org)... 193.62.193.8
    Connecting to ftp.ensembl.org (ftp.ensembl.org)|193.62.193.8|:21... connected.
    Logging in as anonymous ... Logged in!
    ==> SYST ... done.    ==> PWD ... done.
    ==> TYPE I ... done.  ==> CWD (1) /pub/release-98/fasta/mus_musculus/dna ... done.
    ==> SIZE Mus_musculus.GRCm38.dna.primary_assembly.fa.gz ... 805984352
    ==> PASV ... done.    ==> RETR Mus_musculus.GRCm38.dna.primary_assembly.fa.gz ... done.
    Length: 805984352 (769M) (unauthoritative)
    
    Mus_musculus.GRCm38 100%[===================>] 768.65M  3.72MB/s    in 3m 24s  
    
    2020-01-16 19:05:30 (3.76 MB/s) - ‘Mus_musculus.GRCm38.dna.primary_assembly.fa.gz’ saved [805984352]
    
    --2020-01-16 19:05:31--  ftp://ftp.ensembl.org/pub/release-98/gtf/mus_musculus/Mus_musculus.GRCm38.98.gtf.gz
               => ‘Mus_musculus.GRCm38.98.gtf.gz’
    Resolving ftp.ensembl.org (ftp.ensembl.org)... 193.62.193.8
    Connecting to ftp.ensembl.org (ftp.ensembl.org)|193.62.193.8|:21... connected.
    Logging in as anonymous ... Logged in!
    ==> SYST ... done.    ==> PWD ... done.
    ==> TYPE I ... done.  ==> CWD (1) /pub/release-98/gtf/mus_musculus ... done.
    ==> SIZE Mus_musculus.GRCm38.98.gtf.gz ... 30256597
    ==> PASV ... done.    ==> RETR Mus_musculus.GRCm38.98.gtf.gz ... done.
    Length: 30256597 (29M) (unauthoritative)
    
    Mus_musculus.GRCm38 100%[===================>]  28.85M  5.48MB/s    in 5.3s    
    
    2020-01-16 19:05:40 (5.48 MB/s) - ‘Mus_musculus.GRCm38.98.gtf.gz’ saved [30256597]
    
    --2020-01-16 19:05:41--  ftp://ftp.ensembl.org/pub/release-98/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
               => ‘Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz’
    Resolving ftp.ensembl.org (ftp.ensembl.org)... 193.62.193.8
    Connecting to ftp.ensembl.org (ftp.ensembl.org)|193.62.193.8|:21... connected.
    Logging in as anonymous ... Logged in!
    ==> SYST ... done.    ==> PWD ... done.
    ==> TYPE I ... done.  ==> CWD (1) /pub/release-98/fasta/homo_sapiens/dna ... done.
    ==> SIZE Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz ... 881211416
    ==> PASV ... done.    ==> RETR Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz ... done.
    Length: 881211416 (840M) (unauthoritative)
    
    Homo_sapiens.GRCh38 100%[===================>] 840.39M  8.60MB/s    in 2m 58s  
    
    2020-01-16 19:08:43 (4.72 MB/s) - ‘Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz’ saved [881211416]
    
    --2020-01-16 19:08:44--  ftp://ftp.ensembl.org/pub/release-98/gtf/homo_sapiens/Homo_sapiens.GRCh38.98.gtf.gz
               => ‘Homo_sapiens.GRCh38.98.gtf.gz’
    Resolving ftp.ensembl.org (ftp.ensembl.org)... 193.62.193.8
    Connecting to ftp.ensembl.org (ftp.ensembl.org)|193.62.193.8|:21... connected.
    Logging in as anonymous ... Logged in!
    ==> SYST ... done.    ==> PWD ... done.
    ==> TYPE I ... done.  ==> CWD (1) /pub/release-98/gtf/homo_sapiens ... done.
    ==> SIZE Homo_sapiens.GRCh38.98.gtf.gz ... 46712404
    ==> PASV ... done.    ==> RETR Homo_sapiens.GRCh38.98.gtf.gz ... done.
    Length: 46712404 (45M) (unauthoritative)
    
    Homo_sapiens.GRCh38 100%[===================>]  44.55M  6.53MB/s    in 9.3s    
    
    2020-01-16 19:08:57 (4.79 MB/s) - ‘Homo_sapiens.GRCh38.98.gtf.gz’ saved [46712404]
    
    CPU times: user 3.91 s, sys: 853 ms, total: 4.76 s
    Wall time: 6min 56s


### Build the mixed species index

`kb` can build a single transcriptome index with multiple references. The FASTAs and GTFs must be passed in as a comma-separated list.

__Note__: Because Google Colab offers limited RAM, we split the index into 4 parts.


```
%%time
!kb ref -i mixed_index.idx -g mixed_t2g.txt -f1 mixed_cdna.fa -n 4 \
Mus_musculus.GRCm38.dna.primary_assembly.fa.gz,Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz \
Mus_musculus.GRCm38.98.gtf.gz,Homo_sapiens.GRCh38.98.gtf.gz
```

    [2020-01-16 19:09:00,016]    INFO Preparing Mus_musculus.GRCm38.dna.primary_assembly.fa.gz, Mus_musculus.GRCm38.98.gtf.gz
    [2020-01-16 19:09:00,016]    INFO Decompressing Mus_musculus.GRCm38.98.gtf.gz to tmp
    [2020-01-16 19:09:03,821]    INFO Creating transcript-to-gene mapping at /content/tmp/tmp_54cxwsm
    [2020-01-16 19:09:41,067]    INFO Decompressing Mus_musculus.GRCm38.dna.primary_assembly.fa.gz to tmp
    [2020-01-16 19:10:07,073]    INFO Sorting tmp/Mus_musculus.GRCm38.dna.primary_assembly.fa to /content/tmp/tmp8njkjc7p
    [2020-01-16 19:17:20,153]    INFO Sorting tmp/Mus_musculus.GRCm38.98.gtf to /content/tmp/tmpdb4z6qdv
    [2020-01-16 19:18:16,788]    INFO Splitting genome tmp/Mus_musculus.GRCm38.dna.primary_assembly.fa into cDNA at /content/tmp/tmpbwrr1fgf
    [2020-01-16 19:18:16,788] WARNING The following chromosomes were found in the FASTA but doens't have any "transcript" features in the GTF: JH584302.1, GL456394.1, GL456383.1, GL456392.1, GL456393.1, GL456396.1, GL456213.1, GL456366.1, GL456370.1, GL456360.1, JH584300.1, GL456390.1, GL456359.1, GL456387.1, GL456378.1, GL456389.1, GL456379.1, GL456368.1, GL456382.1, GL456367.1, JH584301.1. No sequences will be generated for these chromosomes.
    [2020-01-16 19:19:12,867]    INFO Wrote 142446 cDNA transcripts
    [2020-01-16 19:19:12,870]    INFO Preparing Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz, Homo_sapiens.GRCh38.98.gtf.gz
    [2020-01-16 19:19:12,870]    INFO Decompressing Homo_sapiens.GRCh38.98.gtf.gz to tmp
    [2020-01-16 19:19:19,514]    INFO Creating transcript-to-gene mapping at /content/tmp/tmpbcio2ycd
    [2020-01-16 19:20:16,297]    INFO Decompressing Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz to tmp
    [2020-01-16 19:20:44,907]    INFO Sorting tmp/Homo_sapiens.GRCh38.dna.primary_assembly.fa to /content/tmp/tmpotz6u8im
    [2020-01-16 19:28:55,130]    INFO Sorting tmp/Homo_sapiens.GRCh38.98.gtf to /content/tmp/tmpm1lths39
    [2020-01-16 19:30:22,587]    INFO Splitting genome tmp/Homo_sapiens.GRCh38.dna.primary_assembly.fa into cDNA at /content/tmp/tmpzdmdny6v
    [2020-01-16 19:30:22,587] WARNING The following chromosomes were found in the FASTA but doens't have any "transcript" features in the GTF: KI270311.1, KI270580.1, KI270424.1, KI270304.1, KI270305.1, KI270708.1, KI270591.1, KI270316.1, GL000008.2, KI270507.1, KI270364.1, KI270749.1, KI270548.1, KI270363.1, KI270414.1, KI270709.1, KI270583.1, KI270395.1, KI270322.1, KI270746.1, KI270390.1, KI270435.1, KI270411.1, KI270362.1, KI270751.1, KI270755.1, KI270735.1, GL000214.1, KI270312.1, KI270714.1, KI270317.1, KI270303.1, KI270373.1, KI270753.1, KI270587.1, KI270385.1, KI270425.1, KI270340.1, KI270391.1, KI270590.1, KI270522.1, KI270419.1, KI270730.1, KI270742.1, KI270302.1, KI270740.1, KI270757.1, KI270584.1, KI270723.1, KI270392.1, KI270538.1, KI270732.1, KI270544.1, KI270722.1, KI270736.1, KI270737.1, KI270707.1, KI270528.1, KI270396.1, KI270379.1, KI270374.1, KI270752.1, KI270729.1, KI270335.1, KI270508.1, KI270512.1, KI270371.1, KI270366.1, KI270589.1, KI270382.1, KI270418.1, KI270519.1, KI270510.1, KI270412.1, KI270724.1, KI270310.1, KI270738.1, KI270389.1, KI270745.1, KI270394.1, KI270336.1, GL000221.1, KI270468.1, KI270465.1, KI270579.1, KI270372.1, KI270378.1, KI270725.1, KI270741.1, GL000208.1, GL000224.1, KI270717.1, KI270466.1, KI270376.1, KI270743.1, KI270420.1, KI270715.1, KI270386.1, KI270509.1, KI270329.1, KI270593.1, KI270384.1, KI270712.1, KI270530.1, KI270429.1, KI270581.1, KI270718.1, KI270516.1, KI270582.1, KI270393.1, KI270387.1, GL000226.1, KI270739.1, KI270315.1, KI270710.1, KI270333.1, KI270438.1, KI270337.1, KI270467.1, KI270588.1, KI270334.1, KI270517.1, KI270719.1, KI270448.1, KI270417.1, KI270754.1, KI270720.1, KI270521.1, KI270381.1, KI270422.1, KI270756.1, KI270716.1, KI270515.1, KI270518.1, KI270375.1, KI270320.1, KI270330.1, KI270539.1, KI270529.1, KI270511.1, KI270388.1, KI270748.1, KI270747.1, KI270383.1, KI270706.1, KI270338.1, KI270423.1. No sequences will be generated for these chromosomes.
    [2020-01-16 19:31:39,720]    INFO Wrote 227368 cDNA transcripts
    [2020-01-16 19:31:39,725]    INFO Concatenating 2 transcript-to-gene mappings to mixed_t2g.txt
    [2020-01-16 19:31:40,026]    INFO Concatenating 2 cDNAs to mixed_cdna.fa
    [2020-01-16 19:31:41,876]    INFO Splitting mixed_cdna.fa into 4 parts
    [2020-01-16 19:31:46,079]    INFO Indexing /content/tmp/tmp5sr_pily to mixed_index.idx.0
    [2020-01-16 19:38:23,368]    INFO Indexing /content/tmp/tmpgxjc_cx6 to mixed_index.idx.1
    [2020-01-16 19:44:32,316]    INFO Indexing /content/tmp/tmpj4tfo0m8 to mixed_index.idx.2
    [2020-01-16 19:51:03,185]    INFO Indexing /content/tmp/tmp8wqyvg2b to mixed_index.idx.3
    CPU times: user 13.5 s, sys: 2.6 s, total: 16.1 s
    Wall time: 48min 39s


### Generate an RNA count matrix in H5AD format

The following command will generate an RNA count matrix of cells (rows) by genes (columns) in H5AD format, which is a binary format used to store [Anndata](https://anndata.readthedocs.io/en/stable/) objects. Notice we are providing the index and transcript-to-gene mapping we downloaded in the previous step to the `-i` and `-g` arguments respectively. Also, these reads were generated with the 10x Genomics Chromium Single Cell v2 Chemistry, hence the `-x 10xv2` argument. To view other supported technologies, run `kb --list`.

__Note:__ If you would like a Loom file instead, replace the `--h5ad` flag with `--loom`. If you want to use the raw matrix output by `kb` instead of their H5AD or Loom converted files, omit these flags.


```
%%time
!kb count -i mixed_index.idx.0,mixed_index.idx.1,mixed_index.idx.2,mixed_index.idx.3 \
-g mixed_t2g.txt -x 10xv3 -o output --h5ad -t 2 \
1k_hgmm_v3_S1_L001_R1_001.fastq.gz 1k_hgmm_v3_S1_L001_R2_001.fastq.gz \
1k_hgmm_v3_S1_L002_R1_001.fastq.gz 1k_hgmm_v3_S1_L002_R2_001.fastq.gz \
1k_hgmm_v3_S1_L003_R1_001.fastq.gz 1k_hgmm_v3_S1_L003_R2_001.fastq.gz \
1k_hgmm_v3_S1_L004_R1_001.fastq.gz 1k_hgmm_v3_S1_L004_R2_001.fastq.gz
```

    [2020-01-16 19:57:50,686]    INFO Generating BUS file using 4 indices
    [2020-01-16 19:57:50,686]    INFO Generating BUS file to output/tmp/bus_part0 from
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L001_R1_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L001_R2_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L002_R1_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L002_R2_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L003_R1_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L003_R2_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L004_R1_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO         1k_hgmm_v3_S1_L004_R2_001.fastq.gz
    [2020-01-16 19:57:50,686]    INFO Using index mixed_index.idx.0
    [2020-01-16 20:14:53,228]    INFO Generating BUS file to output/tmp/bus_part1 from
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L001_R1_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L001_R2_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L002_R1_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L002_R2_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L003_R1_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L003_R2_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L004_R1_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO         1k_hgmm_v3_S1_L004_R2_001.fastq.gz
    [2020-01-16 20:14:53,228]    INFO Using index mixed_index.idx.1
    [2020-01-16 20:30:59,826]    INFO Generating BUS file to output/tmp/bus_part2 from
    [2020-01-16 20:30:59,826]    INFO         1k_hgmm_v3_S1_L001_R1_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO         1k_hgmm_v3_S1_L001_R2_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO         1k_hgmm_v3_S1_L002_R1_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO         1k_hgmm_v3_S1_L002_R2_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO         1k_hgmm_v3_S1_L003_R1_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO         1k_hgmm_v3_S1_L003_R2_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO         1k_hgmm_v3_S1_L004_R1_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO         1k_hgmm_v3_S1_L004_R2_001.fastq.gz
    [2020-01-16 20:30:59,827]    INFO Using index mixed_index.idx.2
    [2020-01-16 20:53:38,011]    INFO Generating BUS file to output/tmp/bus_part3 from
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L001_R1_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L001_R2_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L002_R1_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L002_R2_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L003_R1_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L003_R2_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L004_R1_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO         1k_hgmm_v3_S1_L004_R2_001.fastq.gz
    [2020-01-16 20:53:38,012]    INFO Using index mixed_index.idx.3
    [2020-01-16 21:15:37,278]    INFO Merging BUS records to output from
    [2020-01-16 21:15:37,278]    INFO         output/tmp/bus_part0
    [2020-01-16 21:15:37,278]    INFO         output/tmp/bus_part1
    [2020-01-16 21:15:37,278]    INFO         output/tmp/bus_part2
    [2020-01-16 21:15:37,278]    INFO         output/tmp/bus_part3
    [2020-01-16 21:17:58,873]    INFO Sorting BUS file output/output.bus to output/tmp/output.s.bus
    [2020-01-16 21:19:58,502]    INFO Whitelist not provided
    [2020-01-16 21:19:58,502]    INFO Copying pre-packaged 10XV3 whitelist to output
    [2020-01-16 21:19:59,491]    INFO Inspecting BUS file output/tmp/output.s.bus
    [2020-01-16 21:20:52,633]    INFO Correcting BUS records in output/tmp/output.s.bus to output/tmp/output.s.c.bus with whitelist output/10xv3_whitelist.txt
    [2020-01-16 21:22:16,890]    INFO Sorting BUS file output/tmp/output.s.c.bus to output/output.unfiltered.bus
    [2020-01-16 21:24:06,413]    INFO Generating count matrix output/counts_unfiltered/cells_x_genes from BUS file output/output.unfiltered.bus
    [2020-01-16 21:24:42,940]    INFO Reading matrix output/counts_unfiltered/cells_x_genes.mtx
    [2020-01-16 21:24:57,663]    INFO Writing matrix to h5ad output/counts_unfiltered/adata.h5ad
    CPU times: user 28.2 s, sys: 3.99 s, total: 32.1 s
    Wall time: 1h 27min 16s


## Analysis

See [this notebook](https://github.com/pachterlab/MBGBLHGP_2019/blob/master/Supplementary_Figure_6_7/analysis/hgmm10k_v3_single_gene.Rmd) for how to process and load count matrices for a species mixing experiment.


```

```