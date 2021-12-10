xquery version "1.0-ml";

for $x in fn:doc()
  return
    xdmp:spawn-function(function() {    
    for $auth at $a in $x//AuthorList/Author/LastName/data()
      return
        for $one in $x
          let $lname := $x//AuthorList/Author[$a]/LastName/data()
          let $fname :=  $x//AuthorList/Author[$a]/ForeName/data()

          let $search-result:= cts:search(doc(), cts:and-query((
          cts:element-value-query(xs:QName("LastName"), $lname),
          cts:element-value-query(xs:QName("ForeName"), $fname)
          )))
          let $fullName := concat($fname, ' ', $lname)
          let $uri := concat($fname, '-', $lname)
          
          let $root := <AuthorRecord>
          <FullName>{$fullName}</FullName>
          <AffiliationList>
          {for $affiliation in $search-result
          return
            for $aff in $affiliation
          return <Affiliation>{fn:distinct-values($aff//AuthorList/Author/AffiliationInfo/Affiliation/data())}</Affiliation>
          }
          </AffiliationList>
          <CoAuthor>
            {for $CoAuthor in $search-result
            return
            for $i at $b in $CoAuthor//AuthorList/Author/LastName/data()
              return
            for $f in $CoAuthor
              let $laname := $CoAuthor//AuthorList/Author[$b]/LastName/data()
              let $foname :=  $CoAuthor//AuthorList/Author[$b]/ForeName/data()
              let $name := concat($foname, ' ', $laname)
          return <Author>{fn:distinct-values($name[not(.=$fullName)])}</Author>
          }

          </CoAuthor>

          <GrantList>
          {for $grant in $search-result
          return <Grant><GrantDetails>{$grant//GrantList/Grant/GrantID/data()}</GrantDetails></Grant>
          }
          </GrantList>

          <PublicationList>
            <first-publication> 
            {for $article in $search-result 
               let $min := min($search-result//Article/ArticleDate/Year)
             return $article//Article/ArticleDate[Year=$min]/Year/text()}
            </first-publication>
            <last-publication>
              {for $article in $search-result 
                 let $max := max($search-result//Article/ArticleDate/Year)
               return $article//Article/ArticleDate[Year=$max]/Year/text()}
            </last-publication>
          {for $article in $search-result
          return
            <Article>
               <ArticleTitle>
               {for $title in $article
                return $title//Article/ArticleTitle/data()}
              </ArticleTitle>
              <JournalTitle>{for $journalTitle in $article
                return $journalTitle//Article/Journal/Title/data()}
              </JournalTitle>
              <Year>{for $year in $article
                return $year//Article/ArticleDate/Year/data()}
              </Year>
              <Authors>
              {for $auth in $article
            return
            for $i at $a in $auth//AuthorList/Author/LastName/data()
              return
            for $f in $auth
              let $lname := $auth//AuthorList/Author[$a]/LastName/data()
              let $fname :=  $auth//AuthorList/Author[$a]/ForeName/data()
              let $name := concat($fname, ' ', $lname)
          return <Author>{fn:distinct-values($name[not(.=$fullName)])}</Author>
          }
          </Authors>
            </Article>
          }
          </PublicationList>
          </AuthorRecord>

          return xdmp:document-insert(
              $uri, $root,
              <options xmlns="xdmp:document-insert">  
                <permissions>{xdmp:default-permissions()}</permissions>
                <collections>{
                  <collection>/my/additional/collection</collection>,
                  for $coll in xdmp:default-collections()
                  return <collection>{$coll}</collection>
                }</collections>
              </options>)})