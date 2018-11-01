
<body>
    <h1>Course Project</h1>
    <p>Scraping a local job site for data scientist jobs in HK</p>
</body>


```python
import requests
import bs4
from bs4 import BeautifulSoup
import pandas as pd
import time

import re
import pandas as pd
import math
```


```python
def get_job_text(job_link):
    _page = requests.get(job_link)
    _soup = BeautifulSoup(_page.text, "html.parser")
    #print(job_link)
    x = _soup.find(name="div", attrs={"class":"jobsearch-JobComponent-description"})
    if x:
        return x.text
    else:
        return "Not Found"
```


```python
#URL = "https://www.indeed.hk/viewjob?jk=2dfb473af85953b5"
#URL = "https://www.indeed.hk/viewjob?jk=6567706422d33abe"
#get_job_text(URL)
```


```python
start = 0
end = 100

columns = ["job_title", "company_name", "details_link", "job_text"]
jobs_df = pd.DataFrame(columns = columns)


while True:
    url = "https://www.indeed.hk/jobs?q=%22data+scientist%22&start=" + str(start)
    time.sleep(1)  # ensuring at least 1 second between page grabs
    
    page = requests.get(url)
    soup = BeautifulSoup(page.text, "html.parser")
    
    # how many jobs there are?
    if start == 0:
        # a loop here is silly I know
        for div in soup.find_all(name="meta" , attrs={"name":"description"}):
            matchObj = re.match( r"^([0-9]+)", div['content'])
            num_jobs = matchObj.group()
            end = math.floor((int(num_jobs)-1)/10) * 10
            # data frame will contain more records, maybe because of duplicated sponsored jobs
            # and also our last page contains more jobs than it should.
            print("There are ", num_jobs, "jobs.")
            print("start index for sites:", start, "ending index:", end)
            #end = 20

    print(url)
    start = start + 10
    # somehow the main loop, for each single job per page (should be 16 jobs, but seems not always)
    for div in soup.find_all(name="div", attrs={"class":"row"}):
        num = (len(jobs_df) + 1)
        job_post = []
        
        # job title
        for a in div.find_all(name="a", attrs={"data-tn-element":"jobTitle"}):
            job_post.append(a["title"])
        
        # job company
        company = div.find_all(name="span", attrs={"class":"company"})
        if len(company) > 0:
            for b in company:
                job_post.append(b.text.strip())
        else:
            sec_try = div.find_all(name="span", attrs={"class":"result-link-source"})
            for span in sec_try:
                job_post.append(span.text.strip())

        job_link = "https://www.indeed.hk/viewjob?jk=" + div["data-jk"]
        job_post.append(job_link)
        job_text = get_job_text(job_link)
        
        #job_post.append(job_text[0:10])
        job_post.append(job_text)
        
        #print(job_post)
        jobs_df.loc[num] = job_post
        
    if start > end:
        break
    
```

    There are  79 jobs.
    start index for sites: 0 ending index: 70
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=0
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=10
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=20
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=30
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=40
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=50
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=60
    https://www.indeed.hk/jobs?q=%22data+scientist%22&start=70
    

Save this now to a file. Further processing will be done in another script.


```python
jobs_df.to_csv("ds_jobs_indeed_v01_20181101.csv", index=False)
```


```python
jobs_df.head()
#print(jobs_df.iloc[0,3])
# need to check for duplcates, there are a lot.
sum(jobs_df.duplicated())

```




    48




```python
jobs_df.shape #(128, 4)

```




    (128, 4)



END OF SCRIPT

The rest is templates.


```python
#URL = "https://www.indeed.com/jobs?q=data+scientist+%2420%2C000&l=New+York&start=10"
#URL = "https://www.indeed.hk/jobs?q=%22data+scientist%22&l="
URL = "https://www.indeed.hk/jobs?q=%22data+scientist%22"
URL = "https://www.indeed.hk/viewjob?jk=2dfb473af85953b5"
URL = "https://www.indeed.hk/viewjob?jk=6567706422d33abe"



#    https://www.indeed.hk/jobs?q=%22data+scientist%22&l=
#conducting a request of the stated URL above:
page = requests.get(URL)
#specifying a desired format of “page” using the html parser - this allows python to read the various components of the page, rather than treating it as one long string.
soup = BeautifulSoup(page.text, "html.parser")
#printing soup in a more structured tree format that makes for easier reading
#print(soup.prettify())

#for x in soup.find_all(name="div", attrs={"class":"jobsearch-JobComponent-description"}):
#    print(x.text)
#with open("indeed_html.txt", "w") as text_file:
#    text_file.write(soup.prettify())
x = soup.find(name="div", attrs={"class":"jobsearch-JobComponent-description"})
#print(x.text)



```


```python
def extract_job_title_from_result(soup):
    jobs = []
    for div in soup.find_all(name="div", attrs={"class":"row"}):
        #print(div)
        for a in div.find_all(name="a", attrs={"data-tn-element":"jobTitle"}):
            print(a)
            jobs.append(a["title"])
    return(jobs)
extract_job_title_from_result(soup)
```

    <a class="jobtitle turnstileLink" data-tn-element="jobTitle" href="/pagead/clk?mo=r&amp;ad=-6NYlbfkN0C1KiKFNLSTeoagfKR_iBMdGU9SgqzP7Goclj1knC2rZz8RByLEz8BsgzjaLoQa0qioYCsrfsEiWnLOOivI-1CXyZ5ODDVkudJreSEG7GCpk_65j_UwhX55pdMnIfYmIbURZ9FMTtCdSjmLBdfn_PR-hK3fgyoD5pGFlu32lqxNK08UTHGSVYzVzpRgic1AkwcxgBTVruRB8v17rhC1qiV40EcriOabpzQe5Ld8SpQKQyxGQyq6B1Q3hNUzTdIkniHL0Y5kaIj8npodWAf39v4UbB8S9hmhyDlOkfMPzR40HEI4AwNOWeOdhbleUheBuyqv4NEdjaSljhDZ62Ki1MRXUufTPXdByccVVlCQnyZJZ9O0s8eXTbuQIYWdImXCKMygLtp5pM96pdOPBwePE7pl2uSbK-Ky2A9MDm7d-ii0Uc1FbSjB9bpwnejreH5HQ5HKp9kt2RRjEVs6S7My_vwppyZTByqVIaqlPUrgmp04FBc3u2S_1fH2qf4amUfh-nPgQHHddjgDbhGCo5D4k9OTcUXqKwHUAoCQhdLpAIOGPbLHsaoctz2oLz7U9y64N-osxnmzlrA-Iwwn4NUwTNrhiBYEIknbXIII9y2cgHkkr7GiLOSs_yrvaJbm9JEGsqU6mTbU9VaNnvgXCbFBcMGivXgww_m1Z4Uiicp4S3dwc6kJvGmYX4DQ7wda_rJM5jA=&amp;vjs=3&amp;p=1&amp;sk=&amp;fvj=0" id="sja1" onclick="setRefineByCookie([]); sjoc('sja1',0); convCtr('SJ')" onmousedown="sjomd('sja1'); clk('sja1');" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="jobtitle turnstileLink" data-tn-element="jobTitle" href="/pagead/clk?mo=r&amp;ad=-6NYlbfkN0C1KiKFNLSTeoagfKR_iBMdGU9SgqzP7Goclj1knC2rZz8RByLEz8BsgzjaLoQa0qgaiv3bcZJOpbLSc2mJtxwWA9VNJhts16TnibGUfimIpiOLDmZnFgcm_L2c6IIBR7U5XxNy2x9KOmLtKObsD_JlDuuJM79gGxmR-X4g2MugbobOSaxv9df863wFiSKKNsbXTox3vjAV9wZU2eVeQTnbKgiHjkSSE0cY6iG-6YhUC9dMlu_UqdrNb9InW8nw05NRhsh8sKqtlzMeL4V-sqUcHKbx-CrnJz76fJUk1qY3PesBqeQIqLbMrA-twEk7OUCCzmV_ao1KO0T5uDJg4iytxM3jB75IZ1oHSPf5s92_NvSgpGKgTCMo33y2GcucPlUmYjlNJgiXMbqEF5ua1yj-Fma8B6vyDpT1sCQU08WPkWbqPA5KfwhaJNty7vTw2I0UWXFwLHEHgt_Rzc27LymcrarKRKLWLYo6EmCbRMLbwXPRFbI65f4mRG1pEIixzicqhnT3Y4tkloxu8DxhWkwQ5ojyRhE4ly2dQ7KiFgudNgzx4ArDHAS7Ul-dkWfVpwdtlgn0N2-Zxyer1sOCTKdCM89b6wt2WH5bn-gxA8vU_Xf8H1jxUSeLgXWpmBdFbV5I06lc68cjIwH9hOZqvkeUn6MH2uK9wMc8FW40kr0pd99fzlusG9iv&amp;vjs=3&amp;p=2&amp;sk=&amp;fvj=0" id="sja2" onclick="setRefineByCookie([]); sjoc('sja2',0); convCtr('SJ')" onmousedown="sjomd('sja2'); clk('sja2');" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="jobtitle turnstileLink" data-tn-element="jobTitle" href="/pagead/clk?mo=r&amp;ad=-6NYlbfkN0C1KiKFNLSTeoagfKR_iBMdGU9SgqzP7Goclj1knC2rZz8RByLEz8BsgzjaLoQa0qhxhTKY1Jp5Dg3tFhOkACFWrOGWboh-NMQADnzLrvIS6pkKXPgn3ze4Jm1jt-5pf2qeJ_S580DymC5AsQO_ooWN1OM0elqdAeSCVknW3wJyz0J1-6mhabbcL22zEm33HhQ22wAA5IzC6DwHqe0FH8s9HPbFG6jf_SrpF3uM25zroyu4_LY_FtU4An38KZKIv47hsWoFfSh6oHOwmNZE046vu0idCBIUdtg9N65-qPttJjC9AIhq8_G5leDn5SFecBwHQiARmNbLK5-wTZf17VpTD3235O6hH_pXAw3cD2daDhGGyyLQXMqkyNoCHnWBwu3dMvRJRb5DiMAOrnS6r1o_wqF8QDOFUt1fJcyFZzw3YfN5DJL0zFDycoNVP4F3qc3_SQbGpkDrQ_1OoMxsG8jcqaJXZjeXNJtIe-LtDPFiBJGndnOUFI3vl60rIvzzFhaxR_X0gV9z3Usn29EuoTo81T_ko6EA1BieEyNR2jCfkH_6Dr0xnMz8C7hS832bsNSwiU5VGb2Pgp__3vLIBjqGh4S8dfbkz32_c0q3pcAuKZpzsISueFQNmhNpyGqCfvSEYc99LEDyTEAgsr94M-rdOxjhjY7ZGm9twozOziYKkKXd4XUY7Ate&amp;vjs=3&amp;p=3&amp;sk=&amp;fvj=0" id="sja3" onclick="setRefineByCookie([]); sjoc('sja3',0); convCtr('SJ')" onmousedown="sjomd('sja3'); clk('sja3');" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=23768ba7988bcd78&amp;fccid=683f1fb26dcffb4c&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[0],true,0);" onmousedown="return rclk(this,jobmap[0],0);" rel="noopener nofollow" target="_blank" title="Supply Chain Data Scientist">Supply Chain <b>Data</b> <b>Scientist</b></a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=c1c83d3135356139&amp;fccid=55c55b2c488e1a45&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[1],true,0);" onmousedown="return rclk(this,jobmap[1],0);" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=2011973ef100418c&amp;fccid=38c0d5e4f2a99768&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[2],true,0);" onmousedown="return rclk(this,jobmap[2],0);" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=d1e710d3d5a4d45b&amp;fccid=7205cf60d362cea1&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[3],true,0);" onmousedown="return rclk(this,jobmap[3],0);" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=25caad1ec1d1ff7a&amp;fccid=5d898ab80c3f3e38&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[4],true,0);" onmousedown="return rclk(this,jobmap[4],0);" rel="noopener nofollow" target="_blank" title="Data Scientist / Quantitative Analyst"><b>Data</b> <b>Scientist</b> / Quantitative Analyst</a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=5fb4aa72442e34e1&amp;fccid=55bf6ffe5c42e816&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[5],true,0);" onmousedown="return rclk(this,jobmap[5],0);" rel="noopener nofollow" target="_blank" title="Data Scientist - Hong Kong"><b>Data</b> <b>Scientist</b> - Hong Kong</a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=7c4e3ed4b7d830af&amp;fccid=f1943ec5b4031bab&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[6],true,0);" onmousedown="return rclk(this,jobmap[6],0);" rel="noopener nofollow" target="_blank" title="Data Scientist / Computational Scientist"><b>Data</b> <b>Scientist</b> / Computational <b>Scientist</b></a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=af25aa05f0edbc76&amp;fccid=afefea0edad5ffb5&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[7],true,0);" onmousedown="return rclk(this,jobmap[7],0);" rel="noopener nofollow" target="_blank" title="Research Data Scientist /Big Data Engineer,AI Dept">Research <b>Data</b> <b>Scientist</b> /Big <b>Data</b> Engineer,AI Dept</a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=68468f841bbf0da8&amp;fccid=c604b374adac7e5f&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[8],true,0);" onmousedown="return rclk(this,jobmap[8],0);" rel="noopener nofollow" target="_blank" title="Junior Data Scientist">Junior <b>Data</b> <b>Scientist</b></a>
    <a class="turnstileLink" data-tn-element="jobTitle" href="/rc/clk?jk=319296b51921a080&amp;fccid=7e57982cb894da40&amp;vjs=3" onclick="setRefineByCookie([]); return rclk(this,jobmap[9],true,0);" onmousedown="return rclk(this,jobmap[9],0);" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="jobtitle turnstileLink" data-tn-element="jobTitle" href="/pagead/clk?mo=r&amp;ad=-6NYlbfkN0C1KiKFNLSTeoagfKR_iBMdGU9SgqzP7Goclj1knC2rZz8RByLEz8BsgzjaLoQa0qgaiv3bcZJOpWLZzi_7GLThs8_li16jTdY42fd-EdvHSkBysUNKiZtHKqwO0pNTePpN0MpIrqT3K8oMcq6V3WRv1pL8chaBomHJL3lbSS4d5RTlVtbSmQ49Mt_jfGuAY4BtPULgQ5Jddyd-EWEXsZSZB1EuNsTSSCjpt1xT5sd4aZpr7S8k7HpgNk3IEy4Im--MKp9xAjH2qGiWFK5dY9tDIbAzetVqdtfRgIMSWmrR-izztU6oYqZD_s2q3fu2Py2ma2bt4OtWp4EmMez2JBEynrMrVBDgBjfFL9i2g8pMqzVxyTm23yCB8bE8FhO6Var11mH5npDCc6r28Lpii8dfuLHGAjk1HyFiXKZpL8L_B8p5HISau3NguVFYLugmfXbX3vLZ81m8RRCACO7mjbbvVsls1b45qguUKH4kFgch5LUhgLIesCju7cBvz6BXRnnMp49rQspZFEDXdAToAIuXgAXdo5Rma44YAN1w-imhM9wTWqamYBNyDjKXWNxzdnigDv3ShI_kqh7a2lyzdhst57ywQ3rHKkk3FFC4M1O1Gy3PnGwjS9pcHJERZYEgg_mKTjAD-K__6zIxXq8r00qYqRAal8EUP7hcWEDlKgUb8qw_MdBdwIdilTXuttPZ7i0=&amp;vjs=3&amp;p=4&amp;sk=&amp;fvj=0" id="sja4" onclick="setRefineByCookie([]); sjoc('sja4',0); convCtr('SJ')" onmousedown="sjomd('sja4'); clk('sja4');" rel="noopener nofollow" target="_blank" title="Data Scientist - Media"><b>Data</b> <b>Scientist</b> - Media</a>
    <a class="jobtitle turnstileLink" data-tn-element="jobTitle" href="/pagead/clk?mo=r&amp;ad=-6NYlbfkN0C1KiKFNLSTeoagfKR_iBMdGU9SgqzP7Goclj1knC2rZz8RByLEz8BsgzjaLoQa0qgaiv3bcZJOpWLCWMydEfk0DPwEc0D6uO2G2AihscvH80KGXR7lgXGsimXU_uUYQJqHDrwEJc7ZgByyqIJmyqBLFvzDF0NgeOHDrZCU9nF6sOEeJWdKQ-aLsQPMh0tvNT_1pgudk9t4Z0WdCCxfD5NbF10Ym6KGhtPFD90vUcWJvzkUmUMKdY4V7HHU2UhfPoM8rIQwQcLfN4ZHR739rwyb3wL_lkzzEy6Ts-V9_bw2KcsSMIdTHzlOShqEfA6Z7TNkw1uLtqlfntUJq10mKMtSWP3oL61AwGMdahVIQ8Hec74-e9bDoGdyqgygV6HU0NqWaEy3_uvdzbGwlhvYyiu5w8fpvTZekpKuJhndgFfs9pI03xl6geIDmtyiw4HTkqhKhmvrX_Pp2ku9qPkCtQlnh_j3Y5lazVSxWIhqAlxY3tQNPiD44uODPMdpolWwDhm9bpqH9kHk7U8Y93FM7liOLvc7a9PxpXpj9dx2vRoJiAQRGO2fXO79VUwnq_hD3wLzBTqUKIbYpWcYBnv1sSDOA7w9-YuZjvaARwEWuUPFPxksOE8anKZAAYhu9_7I2agMtNzZl43Bc3P3Hee2fx0QtCjMlVoEM4xPnQS_6Uoy4MFGr0Xejdcp&amp;vjs=3&amp;p=5&amp;sk=&amp;fvj=0" id="sja5" onclick="setRefineByCookie([]); sjoc('sja5',0); convCtr('SJ')" onmousedown="sjomd('sja5'); clk('sja5');" rel="noopener nofollow" target="_blank" title="Data Scientist"><b>Data</b> <b>Scientist</b></a>
    <a class="jobtitle turnstileLink" data-tn-element="jobTitle" href="/pagead/clk?mo=r&amp;ad=-6NYlbfkN0C1KiKFNLSTeoagfKR_iBMdGU9SgqzP7Goclj1knC2rZz8RByLEz8BsgzjaLoQa0qi9JupGlNRPmbPm9VeqpzicW8ttJ3z5oSE-gBhHM2fUEpCIZhfMEIx0f1OaICFLPM79LOw0g79HD80xXZIJwlvaGhROu4-COsfo-QoFTy0JeV3vqSiswb9oEDLESn7KYAfXojosmzNHmj6lmcBezkoINprPswqO0Lp4M8P4DxGU0XDmwVWSpRWexPf04X3cQl74Wb-IN8y6UVUmhpWsTv54B27t8htLv8cyrhiyWLJ1h0HceL6m8Y3hdHA-oNux1IuTYYmsjvH2JM_7Mtu3u4VpUwvBynPDqO1HRfFGbIpfKixPx5ucOnroic9bTNO_vymHiylD9KhlffYwY7Qv7YxqhiTAChHUnWYV0YOu0cLQHfEaA5z3OIqofXVSP6IV_jiXVRhAAKHYvkpo1XfJsJinD51OS_GxgMCKlTjgCUAN-bTigsk6bd7H_Ue4rXYDUIn18kRmagHuj-GgP8sBEiGXYRFBBb0-3_3WCSRCcxRjivY_t4q3ehHW90hj6qDGXaCWVMad0T_Ck9dFUREV1UDVn23AYRSh-TFd8tQgj_awWOFjzgoF9iTMwPd18uFrOCf8tIf1Ylx9qZ5xkERdvWnLoeOXiV0P_a0uVhWs3_YQXi0o2e-dkQ_W2LN9YuRylRDI0AdmKex1J0eJ4Jot8miO8_Ibikuz4o8=&amp;vjs=3&amp;p=6&amp;sk=&amp;fvj=0" id="sja6" onclick="setRefineByCookie([]); sjoc('sja6',0); convCtr('SJ')" onmousedown="sjomd('sja6'); clk('sja6');" rel="noopener nofollow" target="_blank" title="Data Scientist / Computational Scientist"><b>Data</b> <b>Scientist</b> / Computational <b>Scientist</b></a>
    




    ['Data Scientist',
     'Data Scientist',
     'Data Scientist',
     'Supply Chain Data Scientist',
     'Data Scientist',
     'Data Scientist',
     'Data Scientist',
     'Data Scientist / Quantitative Analyst',
     'Data Scientist - Hong Kong',
     'Data Scientist / Computational Scientist',
     'Research Data Scientist /Big Data Engineer,AI Dept',
     'Junior Data Scientist',
     'Data Scientist',
     'Data Scientist - Media',
     'Data Scientist',
     'Data Scientist / Computational Scientist']




```python
# https://www.indeed.hk/viewjob?jk=7400947080a2c1ef

def extract_link_for_details(soup):
    link_details = []
    for div in soup.find_all(name="div", attrs={"class":"row"}):
#        print(div["data-jk"])
        link_details.append("https://www.indeed.hk/viewjob?jk=" + div["data-jk"])
    return(link_details)

extract_link_for_details(soup)
```




    ['https://www.indeed.hk/viewjob?jk=8fac5bf3bd04d950',
     'https://www.indeed.hk/viewjob?jk=1ab9ad946450576f',
     'https://www.indeed.hk/viewjob?jk=f1621540076157c1',
     'https://www.indeed.hk/viewjob?jk=6567706422d33abe',
     'https://www.indeed.hk/viewjob?jk=d92b5c3ca5f3d1f7',
     'https://www.indeed.hk/viewjob?jk=2dfb473af85953b5',
     'https://www.indeed.hk/viewjob?jk=b3b7b9e98a61c1fa',
     'https://www.indeed.hk/viewjob?jk=8065aee734f99b7c',
     'https://www.indeed.hk/viewjob?jk=dec891350591d503',
     'https://www.indeed.hk/viewjob?jk=b4ecafcc9ba3d85b',
     'https://www.indeed.hk/viewjob?jk=608c233395777324',
     'https://www.indeed.hk/viewjob?jk=a08880d50ab5b9b2',
     'https://www.indeed.hk/viewjob?jk=65b3be9b13fea89b',
     'https://www.indeed.hk/viewjob?jk=c1c83d3135356139',
     'https://www.indeed.hk/viewjob?jk=a59b9777a43f6a69',
     'https://www.indeed.hk/viewjob?jk=3d6b604aed24017e']




```python
# the details page

URL = "https://www.indeed.hk/viewjob?jk=8fac5bf3bd04d950"
page_details = requests.get(URL)
#specifying a desired format of “page” using the html parser - this allows python to read the various components of the page, rather than treating it as one long string.
soup_details = BeautifulSoup(page_details.text, "html.parser")
#printing soup in a more structured tree format that makes for easier reading
#print(soup_details.prettify())

with open("indeed_details_html.txt", "w") as text_file:
    text_file.write(soup_details.prettify())
```


```python
def extract_company_from_result(soup):
    companies = []
    for div in soup.find_all(name="div", attrs={"class":"row"}):
        company = div.find_all(name="span", attrs={"class":"company"})
        if len(company) > 0:
            for b in company:
                companies.append(b.text.strip())
        else:
            sec_try = div.find_all(name="span", attrs={"class":"result-link-source"})
            for span in sec_try:
                companies.append(span.text.strip())
    return(companies)
 
extract_company_from_result(soup)
```




    ['Terminal 1',
     'Cogs Agency HK',
     'RADICA SYSTEMS LIMITED',
     'BAH Partners',
     'Lab Viso Limited',
     'Cathay Pacific',
     'AXA',
     'Neo Derm Ltd.',
     'IBM',
     'Thinkcol.AI',
     'AUREXIA Consulting',
     'Libbler',
     'Credit Suisse',
     'SOUTH CHINA MORNING POST PUBLISHERS LTD',
     'Hotmob Limited',
     'ELEVATE HONG KONG HOLDINGS LIMITED']




```python
def extract_summary_from_result(soup):
    summaries = []
    spans = soup.findAll('span', attrs={'class': 'summary'})
    for span in spans:
        summaries.append(span.text.strip())
    return(summaries)

extract_summary_from_result(soup)
```




    ['Data Visualization skills. Design data collection, integration and retention requirements. Conduct data analysis and create algorithms to predict probabilities....',
     'They are looking to hire a Data Scientist, Director, to help enhancing such capabilities. Solid experience in data science, machine learning, Natural Language...',
     'Implement data science and data analytic models in common languages. Devise technical data analytic or data science solutions to open business problem....',
     'Experience with Big Data. 2-8 years years of experience working as a quantitative analyst or data scientist in any area....',
     'We are looking for a talented Data Scientist to join our team. Analyse unstructured data (e.g. (i) extract entity from the unstructured data;...',
     'Essential to have strong understanding in database concepts & data modelling, business intelligence disciplines & data management tools....',
     'Data Scientist (180003VK). Big data/ Data Modelling. 3+ years of relevant working experience in data analytics field as statistician / analyst / data scientist....',
     'Undertake processing of structure and unstructured data. Get involves in preparation of data visualization reports and analytical dashboards....',
     'Job Description Do you want to be a consultant? Here at IBM Global Business Services (GBS) we live and breathe a client-first mindset in everything we do. We...',
     'Knowledge in building data solution for questioning answering/dialog system is a plus. Providing assistance with model implementation and integration at all...',
     'What you will be doing as a Data Scientist :. Passionate about data, models and visualizations. Degree in Statistics, Mathematics, Data Science, Engineering,...',
     '5+ years of experience as data scientist. Well established hedge fund is seeking an experienced data scientist to join their team in Hong Kong....',
     'Data Scientist #114944. You have experience in the areas of data analytics, data mining and machine learning. You are passionate about software and data....',
     'Create data tools to help streamline and automate workflows for Data Analytics & Insights team. Enhance data collection processes to include data from 1st and...',
     'Architect, implement and deploy new data models and data processes in production. Enhance data collection processes to include data from 1st and 3rd party...',
     'We are seeking an ambitious Data Scientist to play a leading role on our Data & Analytics team to use data to accelerate business-driven sustainability....']




```python
def extract_salary_from_result(soup):
    salaries = []
    for div in soup.find_all(name="div", attrs={"class":"row"}):
        try:
            salaries.append(div.find('nobr').text)
        except:
            try:
                div_two = div.find(name="div", attrs={"class":"sjcl"})
                div_three = div_two.find("div")
                salaries.append(div_three.text.strip())
            except:
                salaries.append("Nothing_found")
    return(salaries)
extract_salary_from_result(soup)
```




    ['Terminal 1',
     'Cogs Agency HK',
     'RADICA SYSTEMS LIMITED',
     'BAH Partners',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Nothing_found',
     'Hotmob Limited',
     'ELEVATE HONG KONG HOLDINGS LIMITED']


