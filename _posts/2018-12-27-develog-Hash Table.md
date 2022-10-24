---
title: "Devlog: Hash Table 이해"
excerpt_separator: "<!--more-->"
categories:
  - Devlog
tags:
  - Devlog
  - 자료구조
---

Hash Table
==============

해쉬 테이블은 공간을 미리 확보해두고 입력받은 데이터를 해쉬값으로 테이블의 인덱스, 키값으로 넣어 해 주소에 데이터를
담아낸 것, 탐색 알고리즘 

PHP로 Hash Table 구현
--------------------

해당 알고리즘은 직접 구현하지 못하고 누군가 구현한 Hash Table을 가지고와서 코드분석을 하게되었습니다.
알고리즘에 대한 개념파악 후에 새롭게 짤 예정

    <?php
    /**
     * Created by PhpStorm.
     * User: user
     * Date: 2018. 12. 30.
     * Time: 오전 9:30
     */
    
    class Hash
    {
        public $hashtable = array();
        public $hashtable_size;
    
        public function __construct($tablesize)
        {
            if($tablesize) {
                $this->HashTableSize = $tablesize;
            } else {
                print "Unknown file size\n";
                return -1;
            }
        }
    
        public  function __destruct()
        {
            unset($this->hashtable);
        }
    
        public  function  generate_bucket($string)
        {
            for($i=0; $i <= strlen($string); $i++) {
                $hash = ord($string[$i]) + ($hash << 5) - $hash;
            }
            print "".$this->HashTableSize."\n";
            return($hash%$this->HashTableSize);
        }
    
        public  function  add($string, $associated_array)
        {
            $bucket = $this->generate_bucket($string);
    
            $tmp_array = array();
            $tmp_array['string'] = $string;
            $tmp_array['assoc_array'] = $associated_array;
    
            if(!isset($this->HashTable[$bucket])) {
                $this->HashTable[$bucket] = $tmp_array;
            } else {
                if(is_array($this->HashTable[$bucket])) {
                    array_push($this->HashTable[$bucket], $tmp_array);
                } else {
                    $tmp = $this->HashTable[$bucket];
                    $this->HashTable[$bucket] = array();
                    array_push($this->HashTable[$bucket], $tmp);
                    array_push($this->HashTable[$bucket], $tmp_array);
                }
            }
    
        }
    
        public  function  delete($string, $attrname, $attrvalue)
        {
            $bucket = $this->generate_bucket($string);
    
            if(is_null($this->HashTable[$bucket])) {
                return -1;
            } else {
                if(is_array($this->HashTable[$bucket])) {
                    for($x = 0; $x <= sizeof($this->HashTable[$bucket]); $x++) {
                        if(($this->HashTable[$bucket][$x]['string'] == $string) && ($this->HashTable[$bucket][$x]['.$attrname.'] == $attrvalue)) {
                            unset($this->HashTable[$bucket][$x]);
                        }
                    }
                } else {
                    unset($this->HashTable[$bucket][$x]);
                }
            }
            /** everything is OK **/
            return 0;
        }
    
    
        public  function  search($string)
        {
            $resultArray = array();
    
            $bucket = $this->generate_bucket($string);
    
            if(is_null($this->HashTable[$bucket])) {
                return -1;
            } else {
                if(is_array($this->HashTable[$bucket])) {
                    for($x = 0; $x <= sizeof($this->HashTable[$bucket]); $x++) {
                        if(strcmp($this->HashTable[$bucket][$x]['string'], $string) == 0) {
                            array_push($resultArray,$this->HashTable[$bucket][$x]);
                        }
                    }
                } else {
                    array_push($resultArray,$this->HashTable[$bucket]);
                }
            }
    
            return($resultArray);
        }
    
    }
    
    
    
***

    public  function  generate_bucket($string)
    {
       for($i=0; $i <= strlen($string); $i++) {
           $hash = ord($string[$i]) + ($hash << 5) - $hash;
       }
       print "".$this->HashTableSize."\n";
       return($hash%$this->HashTableSize);
    }
    
ord() $string의 글자 하나씩 아스키코드 값으로 변경 << 비트 연산자 왼쪽으로 5 이동한 값을 구해서 각 알파벳에 대한 고유값
유니크한 값을 만들어 bucket의 key값으로 사용 

 
 



