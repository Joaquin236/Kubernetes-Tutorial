## Listar el directorio /root
ls -l
total 60
-rw-r--r--    1 root     root            56 Aug 14 13:46 q1.json
-rw-r--r--    1 root     root           544 Oct 15  2024 q10.json
-rw-r--r--    1 root     root          8474 Oct 15  2024 q11.json
-rw-r--r--    1 root     root            49 Oct 15  2024 q12.json
-rw-r--r--    1 root     root            49 Oct 15  2024 q13.json
-rw-r--r--    1 root     root           148 Oct 15  2024 q2.json
-rw-r--r--    1 root     root           148 Oct 15  2024 q3.json
-rw-r--r--    1 root     root           204 Oct 15  2024 q4.json
-rw-r--r--    1 root     root           540 Oct 15  2024 q5.json
-rw-r--r--    1 root     root           540 Oct 15  2024 q6.json
-rw-r--r--    1 root     root           539 Oct 15  2024 q7.json
-rw-r--r--    1 root     root           544 Oct 15  2024 q8.json
-rw-r--r--    1 root     root           544 Oct 15  2024 q9.json

## Contenido del q1.json
cat q1.json 
{
    "property1": "value1",
    "property2": "value2"
}

## El comando cat q1.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q1.json | jpath '$.property1'
[
  "value2"
]
echo "cat q1.json | jpath '$.property1'" > /root/answer1.sh

## Contenido del q2.json
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

## El comando cat q2.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q2.json | jpath '$.bus'
[
  {
    "color": "white",
    "price": "$120,000"
  }
]
echo "cat q2.json | jpath '$.bus'" > /root/answer2.sh

## Contenido del fichero q3.json
cat q3.json 
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

## El comando cat q3.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q3.json | jpath $.bus.price
[
  "$120,000"
]
echo "cat q3.json | jpath $.bus.price " > /root/answer3.sh

## Contenido de q4.json
cat q4.json
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

## El comando cat q4.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q4.json | jpath $.vehicles.car.price
[
  "$20,000"
]
echo "cat q4.json | jpath $.vehicles.car.price" > /root/answer4.sh

## Contenido del fichero q5.json
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

## El comando cat q5.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q5.json | jpath $.car.wheels
[
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
]
echo "cat q5.json | jpath $.car.wheels" > /root/answer5.sh

## Contenido del fichero q6.json
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
    }
}

## El comando cat q6.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q6.json | jpath $.car.wheels[2]
[
  {
    "model": "KCMDD3435K",
    "location": "rear-right"
  }
]
echo "cat q6.json | jpath $.car.wheels[2]" > /root/answer6.sh

## Contenido del fichero q7.json
cat q7.json 
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
                "model": "JJDH3434KK",
                "location": "rear-left"
            }
        ]
    }
}

## El comando cat q7.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q7.json | jpath $.car.wheels[2].model
[
  "KCMDD3435K"
]
echo "cat q7.json | jpath $.car.wheels[2].model" > /root/answer7.sh

## Contenido del fichero q8.json
cat q8.json 
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

## El comando cat q8.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q8.json | jpath $.employee.payslips
[
  [
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
]
echo "cat q8.json | jpath $.employee.payslips" > /root/answer8.sh

## Contenido del fichero q9.json
cat q9.json 
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

## El comando cat q9.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q9.json | jpath $.employee.payslips[2]
[
  {
    "month": "august",
    "amount": 3400
  }
]
echo "cat q9.json | jpath $.employee.payslips[2]" > /root/answer9.sh

## Contenido del fichero q10.json
cat q10.json 
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

## El comando cat q10.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q10.json | jpath $.employee.payslips[2].amount
[
  3400
]
echo "cat q10.json | jpath $.employee.payslips[2].amount" > /root/answer10.sh

## Contenido del fichero q11.json
cat q11.json 
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

## El comando cat q11.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q11.json | jpath $.prizes[5].laureates[1]
[
  {
    "id": "914",
    "firstname": "Malala",
    "surname": "Yousafzai",
    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
    "share": "2"
  }
]
echo "cat q11.json | jpath $.prizes[5].laureates[1]" > /root/answer11.sh

## Contenido del fichero q12.json
cat q12.json 
[
    "car",
    "bus",
    "truck",
    "bike"
]

## El comando cat q12.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q12.json | jpath $.0
[
  "car"
]
echo "cat q12.json | jpath $.0" > /root/answer12.sh

## Contenido del fichero q13.json
cat q13.json 
[
    "car",
    "bus",
    "truck",
    "bike"
]

## El comando cat q13.json | jpath '$.property1' debe ser encapsulado con echo y redirigirlo a un fichero
cat q13.json | jpath '$[0,3]'
[
  "car",
  "bike"
]
echo "cat q13.json | jpath '$[0,3]'" > /root/answer13.sh

cat > /root/answer13.sh << 'EOF'
cat q13.json | jpath '$[0,3]'
EOF
