##
ls -l
total 52
-rw-r--r--    1 root     root            56 Oct 15  2024 q1.json
-rw-r--r--    1 root     root           148 Oct 15  2024 q2.json
-rw-r--r--    1 root     root           204 Oct 15  2024 q3.json
-rw-r--r--    1 root     root           310 Oct 15  2024 q4.json
-rw-r--r--    1 root     root           540 Oct 15  2024 q5.json
-rw-r--r--    1 root     root          1079 Oct 15  2024 q6.json
-rw-r--r--    1 root     root           544 Oct 15  2024 q7.json
-rw-r--r--    1 root     root          8474 Oct 15  2024 q8.json
-rw-r--r--    1 root     root          8474 Oct 15  2024 q9.json

##
cat q1.json 
{
    "property1": "value1",
    "property2": "value2"
}

##
cat q1.json | jpath $.*
[
  "value1",
  "value2"
]
echo "cat q1.json | jpath $.*" > /root/answer1.sh

##
cat q2.json 
{
    "car": {
        "color": "blue",
        "price": "$20,000"
    },
    "bus": {
        "color": "white",
        "price": "$120,000"
    }
}

##
cat q2.json | jpath $.*.color
[
  "blue",
  "white"
]
echo "cat q2.json | jpath $.*.color" > /root/answer2.sh

##
cat q3.json 
{
    "vehicles": {
        "car": {
            "color": "blue",
            "price": "$20,000"
        },
        "bus": {
            "color": "white",
            "price": "$120,000"
        }
    }
}

##
cat q3.json | jpath $.*.*.price
[
  "$20,000",
  "$120,000"
]
echo "cat q3.json | jpath $.*.*.price" > /root/answer3.sh

##
cat q4.json 
[
    {
        "model": "KDJ39848T",
        "location": "front-right"
    },
    {
        "model": "MDJ39485DK",
        "location": "front-left"
    },
    {
        "model": "KCMDD3435K",
        "location": "rear-right"
    },
    {
        "model": "JJDH34234KK",
        "location": "rear-left"
    }
]

##
cat q4.json | jpath $.*.model
[
  "KDJ39848T",
  "MDJ39485DK",
  "KCMDD3435K",
  "JJDH34234KK"
]
echo "cat q4.json | jpath $.*.model" > /root/answer4.sh

##
cat q5.json 
{
    "car": {
        "color": "blue",
        "price": "$20,000",
        "wheels": [
            {
                "model": "KDJ39848T",
                "location": "front-right"
            },
            {
                "model": "MDJ39485DK",
                "location": "front-left"
            },
            {
                "model": "KCMDD3435K",
                "location": "rear-right"
            },
            {
                "model": "JJDH34234KK",
                "location": "rear-left"
            }
        ]
    }
}

##
cat q5.json | jpath $.car.wheels[*].model
[
  "KDJ39848T",
  "MDJ39485DK",
  "KCMDD3435K",
  "JJDH34234KK"
]
echo "cat q5.json | jpath $.car.wheels[*].model" > /root/answer5.sh

##
cat q6.json 
{
    "car": {
        "color": "blue",
        "price": "$20,000",
        "wheels": [
            {
                "model": "KDJ39848T",
                "location": "front-right"
            },
            {
                "model": "MDJ39485DK",
                "location": "front-left"
            },
            {
                "model": "KCMDD3435K",
                "location": "rear-right"
            },
            {
                "model": "JJDH34234KK",
                "location": "rear-left"
            }
        ]
    },
    "bus": {
        "color": "white",
        "price": "$120,000",
        "wheels": [
            {
                "model": "BBBB39848T",
                "location": "front-right"
            },
            {
                "model": "BBBB9485DK",
                "location": "front-left"
            },
            {
                "model": "BBBB3435K",
                "location": "rear-right"
            },
            {
                "model": "BBBB4234KK",
                "location": "rear-left"
            }
        ]
    }
}

##
cat q6.json | jpath $.*.wheels[*].model
[
  "KDJ39848T",
  "MDJ39485DK",
  "KCMDD3435K",
  "JJDH34234KK",
  "BBBB39848T",
  "BBBB9485DK",
  "BBBB3435K",
  "BBBB4234KK"
]
echo "cat q6.json | jpath $.*.wheels[*].model" > /root/answer6.sh

##
cat q7.json 
{
    "employee": {
        "name": "john",
        "gender": "male",
        "age": 24,
        "address": {
            "city": "edison",
            "state": "new jersey",
            "country": "united states"
        },
        "payslips": [
            {
                "month": "june",
                "amount": 1400
            },
            {
                "month": "july",
                "amount": 2400
            },
            {
                "month": "august",
                "amount": 3400
            }
        ]
    }
}

##
cat q7.json | jpath $.employee.payslips[*].amount
[
  1400,
  2400,
  3400
]
echo "cat q7.json | jpath $.employee.payslips[*].amount" > /root/answer7.sh

##
cat q8.json 
{
    "prizes": [
        {
            "year": "2018",
            "category": "physics",
            "overallMotivation": "\"for groundbreaking inventions in the field of laser physics\"",
            "laureates": [
                {
                    "id": "960",
                    "firstname": "Arthur",
                    "surname": "Ashkin",
                    "motivation": "\"for the optical tweezers and their application to biological systems\"",
                    "share": "2"
                },
                {
                    "id": "961",
                    "firstname": "Gérard",
                    "surname": "Mourou",
                    "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
                    "share": "4"
                },
                {
                    "id": "962",
                    "firstname": "Donna",
                    "surname": "Strickland",
                    "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2018",
            "category": "chemistry",
            "laureates": [
                {
                    "id": "963",
                    "firstname": "Frances H.",
                    "surname": "Arnold",
                    "motivation": "\"for the directed evolution of enzymes\"",
                    "share": "2"
                },
                {
                    "id": "964",
                    "firstname": "George P.",
                    "surname": "Smith",
                    "motivation": "\"for the phage display of peptides and antibodies\"",
                    "share": "4"
                },
                {
                    "id": "965",
                    "firstname": "Sir Gregory P.",
                    "surname": "Winter",
                    "motivation": "\"for the phage display of peptides and antibodies\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2018",
            "category": "medicine",
            "laureates": [
                {
                    "id": "958",
                    "firstname": "James P.",
                    "surname": "Allison",
                    "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
                    "share": "2"
                },
                {
                    "id": "959",
                    "firstname": "Tasuku",
                    "surname": "Honjo",
                    "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2018",
            "category": "peace",
            "laureates": [
                {
                    "id": "966",
                    "firstname": "Denis",
                    "surname": "Mukwege",
                    "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
                    "share": "2"
                },
                {
                    "id": "967",
                    "firstname": "Nadia",
                    "surname": "Murad",
                    "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2018",
            "category": "economics",
            "laureates": [
                {
                    "id": "968",
                    "firstname": "William D.",
                    "surname": "Nordhaus",
                    "motivation": "\"for integrating climate change into long-run macroeconomic analysis\"",
                    "share": "2"
                },
                {
                    "id": "969",
                    "firstname": "Paul M.",
                    "surname": "Romer",
                    "motivation": "\"for integrating technological innovations into long-run macroeconomic analysis\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2014",
            "category": "peace",
            "laureates": [
                {
                    "id": "913",
                    "firstname": "Kailash",
                    "surname": "Satyarthi",
                    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
                    "share": "2"
                },
                {
                    "id": "914",
                    "firstname": "Malala",
                    "surname": "Yousafzai",
                    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2017",
            "category": "physics",
            "laureates": [
                {
                    "id": "941",
                    "firstname": "Rainer",
                    "surname": "Weiss",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "2"
                },
                {
                    "id": "942",
                    "firstname": "Barry C.",
                    "surname": "Barish",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "4"
                },
                {
                    "id": "943",
                    "firstname": "Kip S.",
                    "surname": "Thorne",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2017",
            "category": "chemistry",
            "laureates": [
                {
                    "id": "944",
                    "firstname": "Jacques",
                    "surname": "Dubochet",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                },
                {
                    "id": "945",
                    "firstname": "Joachim",
                    "surname": "Frank",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                },
                {
                    "id": "946",
                    "firstname": "Richard",
                    "surname": "Henderson",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                }
            ]
        },
        {
            "year": "2017",
            "category": "medicine",
            "laureates": [
                {
                    "id": "938",
                    "firstname": "Jeffrey C.",
                    "surname": "Hall",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                },
                {
                    "id": "939",
                    "firstname": "Michael",
                    "surname": "Rosbash",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                },
                {
                    "id": "940",
                    "firstname": "Michael W.",
                    "surname": "Young",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                }
            ]
        }
    ]
}

##
cat q8.json | jpath $.prizes[*].laureates[*].firstname
[
  "Arthur",
  "Gérard",
  "Donna",
  "Frances H.",
  "George P.",
  "Sir Gregory P.",
  "James P.",
  "Tasuku",
  "Denis",
  "Nadia",
  "William D.",
  "Paul M.",
  "Kailash",
  "Malala",
  "Rainer",
  "Barry C.",
  "Kip S.",
  "Jacques",
  "Joachim",
  "Richard",
  "Jeffrey C.",
  "Michael",
  "Michael W."
]
echo "cat q8.json | jpath $.prizes[*].laureates[*].firstname" > /root/answer8.sh

##
cat q9.json 
{
    "prizes": [
        {
            "year": "2018",
            "category": "physics",
            "overallMotivation": "\"for groundbreaking inventions in the field of laser physics\"",
            "laureates": [
                {
                    "id": "960",
                    "firstname": "Arthur",
                    "surname": "Ashkin",
                    "motivation": "\"for the optical tweezers and their application to biological systems\"",
                    "share": "2"
                },
                {
                    "id": "961",
                    "firstname": "Gérard",
                    "surname": "Mourou",
                    "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
                    "share": "4"
                },
                {
                    "id": "962",
                    "firstname": "Donna",
                    "surname": "Strickland",
                    "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2018",
            "category": "chemistry",
            "laureates": [
                {
                    "id": "963",
                    "firstname": "Frances H.",
                    "surname": "Arnold",
                    "motivation": "\"for the directed evolution of enzymes\"",
                    "share": "2"
                },
                {
                    "id": "964",
                    "firstname": "George P.",
                    "surname": "Smith",
                    "motivation": "\"for the phage display of peptides and antibodies\"",
                    "share": "4"
                },
                {
                    "id": "965",
                    "firstname": "Sir Gregory P.",
                    "surname": "Winter",
                    "motivation": "\"for the phage display of peptides and antibodies\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2018",
            "category": "medicine",
            "laureates": [
                {
                    "id": "958",
                    "firstname": "James P.",
                    "surname": "Allison",
                    "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
                    "share": "2"
                },
                {
                    "id": "959",
                    "firstname": "Tasuku",
                    "surname": "Honjo",
                    "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2018",
            "category": "peace",
            "laureates": [
                {
                    "id": "966",
                    "firstname": "Denis",
                    "surname": "Mukwege",
                    "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
                    "share": "2"
                },
                {
                    "id": "967",
                    "firstname": "Nadia",
                    "surname": "Murad",
                    "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2018",
            "category": "economics",
            "laureates": [
                {
                    "id": "968",
                    "firstname": "William D.",
                    "surname": "Nordhaus",
                    "motivation": "\"for integrating climate change into long-run macroeconomic analysis\"",
                    "share": "2"
                },
                {
                    "id": "969",
                    "firstname": "Paul M.",
                    "surname": "Romer",
                    "motivation": "\"for integrating technological innovations into long-run macroeconomic analysis\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2014",
            "category": "peace",
            "laureates": [
                {
                    "id": "913",
                    "firstname": "Kailash",
                    "surname": "Satyarthi",
                    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
                    "share": "2"
                },
                {
                    "id": "914",
                    "firstname": "Malala",
                    "surname": "Yousafzai",
                    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
                    "share": "2"
                }
            ]
        },
        {
            "year": "2017",
            "category": "physics",
            "laureates": [
                {
                    "id": "941",
                    "firstname": "Rainer",
                    "surname": "Weiss",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "2"
                },
                {
                    "id": "942",
                    "firstname": "Barry C.",
                    "surname": "Barish",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "4"
                },
                {
                    "id": "943",
                    "firstname": "Kip S.",
                    "surname": "Thorne",
                    "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
                    "share": "4"
                }
            ]
        },
        {
            "year": "2017",
            "category": "chemistry",
            "laureates": [
                {
                    "id": "944",
                    "firstname": "Jacques",
                    "surname": "Dubochet",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                },
                {
                    "id": "945",
                    "firstname": "Joachim",
                    "surname": "Frank",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                },
                {
                    "id": "946",
                    "firstname": "Richard",
                    "surname": "Henderson",
                    "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
                    "share": "3"
                }
            ]
        },
        {
            "year": "2017",
            "category": "medicine",
            "laureates": [
                {
                    "id": "938",
                    "firstname": "Jeffrey C.",
                    "surname": "Hall",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                },
                {
                    "id": "939",
                    "firstname": "Michael",
                    "surname": "Rosbash",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                },
                {
                    "id": "940",
                    "firstname": "Michael W.",
                    "surname": "Young",
                    "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
                    "share": "3"
                }
            ]
        }
    ]
}

##
cat q9.json | jpath '$.prizes[?(@.year == 2014)].laureates[*].firstname'
[
  "Kailash",
  "Malala"
]
echo "cat q9.json | jpath '$.prizes[?(@.year == 2014)]
.laureates[*].firstname'" > /root/answer9.sh 