R-codes
=======

web-scraping using R

Código de web scraping para google scholar, usando palavras-chaves. 
Com este code eu consegui retirar informações como: Título, Autores, Resumo e número de citações; da página do google scholar.
O código vou baseado em informações obtidas pelos seguintes R programmers: Kay Cichini, Gabor Pozsgai e Rogério Barbosa.

library(RSelenium)
library(xlsx)

checkForServer() #baixando um servidor do Selenium (so precisa fazer uma vez)
startServer() # mantenha essa janela aberta
Tenha o Mozilla firefox instalado!! Vamos abri-lo:

firefox_con <- remoteDriver(remoteServerAddr = "localhost", 
                            port = 4444, 
                            browserName = "firefox"
)
Abrindo o firefox (a navegacao vai se dar nele)

firefox_con$open() # mantenha essa janela aberta
Realiza o Scrapping

url <- paste("http://scholar.google.com/scholar?q=", "+key+word", "&num=1&as_sdt=1&as_vis=1", 
             sep = "")

firefox_con$navigate("http://scholar.google.com.br")
busca <- firefox_con$findElement(using = "css selector", value = "#gs_hp_tsi")
Keyword <- busca$sendKeysToElement(list("key word", key="enter"))

pages.max <- 10

scraper_internal <- function(x) {
  doc <- htmlParse(url, encoding="UTF-8")
  tit <- xpathSApply(x, "//h3[@class='gs_rt']", xmlValue)
  aut <- xpathSApply(x, "//div[@class='gs_a']", xmlValue)
  abst <- xpathSApply(x, "//div[@class='gs_rs']", xmlValue)
  others <- xpathSApply(x, "//div[@class='gs_fl']", xmlValue)
  dat <- data.frame(TITLE = tit, AUTHORS = aut, ABSTRACT = abst, CITED = others)
}

for (i in seq(1,pages.max*10,10)){
    baseURL <- paste("http://scholar.google.com/scholar?start=", i, "&q=", "+key+word",
                   "&hl=en&lr=lang_en&num=10&as_sdt=1&as_vis=1",
                   sep = "")
  firefox_con$navigate(baseURL)
  pagina <- xmlRoot(htmlParse(
    unlist(firefox_con$getPageSource())
  ))
  result <- scraper_internal(pagina)
  write.xlsx(result, "C:/KEYWORD.xlsx", 
             sheetName = paste("keyword", i), row.names=TRUE, col.names = TRUE, append=TRUE)
