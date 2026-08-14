## Contenido del fichero input.json
cat input.json 
[
  "Apple",
  "Google",
  "Microsoft",
  "Amazon",
  "Facebook",
  "Coca-Cola",
  "Samsung",
  "Disney",
  "Toyota",
  "McDonald's"
]

## Con el comando cat input.json | jpath 'query', redirige el contenido a un fichero
cat input.json | jpath $[0]
[
  "Apple"
]
echo "cat input.json | jpath $[0]" > /root/answer1.sh

## Con el comando cat input.json | jpath 'query', redirige el contenido a un fichero
cat input.json | jpath '$[0:6:4]'
[
  "Apple",
  "Facebook"
]
cat > /root/answer2.sh << 'EOF'
cat input.json | jpath '$[0:6:4]'
EOF

##
cat input.json | jpath '$[0:5]'
[
  "Apple",
  "Google",
  "Microsoft",
  "Amazon",
  "Facebook"
]
cat > /root/answer3.sh << 'EOF'
cat input.json | jpath '$[0:5]'
EOF

##
cat input.json | jpath '$[2:7]'
[
  "Microsoft",
  "Amazon",
  "Facebook",
  "Coca-Cola",
  "Samsung"
]
cat > /root/answer4.sh << 'EOF'
cat input.json | jpath '$[2:7]'
EOF

##
cat input.json | jpath '$[-1:]'
[
  "McDonald's"
]
cat > /root/answer5.sh << 'EOF'
cat input.json | jpath '$[-1:]'
EOF

##
cat input.json | jpath '$[6:]'
[
  "Samsung",
  "Disney",
  "Toyota",
  "McDonald's"
]
cat > /root/answer6.sh << 'EOF'
cat input.json | jpath '$[6:]'
EOF

##
cat input.json | jpath '$[3:-1]'
[
  "Amazon",
  "Facebook",
  "Coca-Cola",
  "Samsung",
  "Disney",
  "Toyota"
]
cat > /root/answer7.sh << 'EOF'
cat input.json | jpath '$[3:-1]'
EOF

##
cat input.json 
[
  "Curabitur Vel Lectus Limited",
  "Libero Morbi Accumsan Industries",
  "Faucibus Ltd",
  "Eu Corp.",
  "Neque Sed Corporation",
  "Nunc Commodo Incorporated",
  "Taciti Sociosqu Industries",
  "Rutrum Lorem Corp.",
  "Proin Corp.",
  "Dolor Fusce Corporation",
  "Malesuada Ut Sem LLC",
  "Mattis Ornare PC",
  "Pede Nonummy Ut LLP",
  "Aliquam PC",
  "Eu Consulting",
  "Leo Morbi Neque Incorporated",
  "Suspendisse Institute",
  "In Tincidunt Congue Consulting",
  "Ipsum Inc.",
  "Nulla Aliquet Proin Consulting",
  "Lorem Luctus Ut Consulting",
  "Sed Sapien Nunc Associates",
  "Feugiat Tellus Industries",
  "Sem LLP",
  "Aliquam Enim Nec Inc.",
  "Feugiat Tellus PC",
  "Dis Parturient Limited",
  "Sed Dictum Corporation",
  "Eu PC",
  "Tellus Faucibus Leo Corp.",
  "Velit Company",
  "Mauris Aliquam Eu Corp.",
  "Rutrum Justo Praesent Industries",
  "Malesuada Fames Associates",
  "Vitae Sodales Foundation",
  "Amet Company",
  "Dignissim Tempor Limited",
  "Morbi Tristique Corp.",
  "Nisi Cum Sociis Foundation",
  "Donec Vitae Erat Incorporated",
  "Fringilla Industries",
  "Elit Incorporated",
  "Velit In Institute",
  "Odio A Purus Incorporated",
  "Hendrerit Id Institute",
  "Aliquet Magna Associates",
  "Dictum Eu Corporation",
  "Integer Mollis Integer Corp.",
  "Libero Proin Mi Foundation",
  "Purus Sapien Associates",
  "Fringilla Associates",
  "Ante Company",
  "Bibendum Company",
  "Convallis Ligula Donec Industries",
  "Elit Inc.",
  "Scelerisque Foundation",
  "Curae; Corp.",
  "Ornare PC",
  "Ut Ipsum Consulting",
  "Tortor Ltd",
  "Convallis Dolor Quisque Foundation",
  "Feugiat Metus Sit Corp.",
  "Nec Orci Incorporated",
  "Arcu Vel Institute",
  "Diam Duis Corp.",
  "Ut Cursus Luctus Incorporated",
  "Vitae LLP",
  "Sed Sem Company",
  "Pede Cum Ltd",
  "Laoreet Lectus Foundation",
  "Semper Dui Foundation",
  "Odio A Purus Inc.",
  "Rutrum Magna Cras PC",
  "A Felis Company",
  "Libero Et Tristique Incorporated",
  "Odio Etiam Associates",
  "Cum Sociis Natoque Industries",
  "Nulla Dignissim Maecenas Inc.",
  "Malesuada Incorporated",
  "Lorem Eu Metus Foundation",
  "In Company",
  "Class Aptent Incorporated",
  "Ac Arcu Nunc Institute",
  "Aliquet Molestie LLP",
  "Sed LLC",
  "Pede LLP",
  "Ante Ipsum Primis Corporation",
  "Eu Dolor Ltd",
  "A Aliquet Consulting",
  "Lacinia Limited",
  "Pretium Aliquet Limited",
  "Magna Nec Corp.",
  "Egestas Corporation",
  "Est Congue Associates",
  "Non Cursus Inc.",
  "Elit Fermentum Associates",
  "Consectetuer Adipiscing Elit Limited",
  "Accumsan Convallis PC",
  "In Ltd"
]

##
cat input.json | jpath '$[-1:]'
[
  "In Ltd"
]
cat > /root/answer8.sh << 'EOF'
cat input.json | jpath '$[-1:]'
EOF

##
cat input.json | jpath '$[-3:]'
[
  "Consectetuer Adipiscing Elit Limited",
  "Accumsan Convallis PC",
  "In Ltd"
]
cat > /root/answer9.sh << 'EOF'
cat input.json | jpath '$[-3:]'
EOF

##
cat input.json | jpath '$[-8:-2]'
[
  "Magna Nec Corp.",
  "Egestas Corporation",
  "Est Congue Associates",
  "Non Cursus Inc.",
  "Elit Fermentum Associates",
  "Consectetuer Adipiscing Elit Limited"
]
cat > /root/answer10.sh << 'EOF'
cat input.json | jpath '$[-8:-2]'
EOF

##
cat input.json 
[
  {
    "age": 35,
    "name": "Tameka Lane",
    "gender": "female",
    "phone": "+1 (850) 469-2827"
  },
  {
    "age": 26,
    "name": "Kristy Day",
    "gender": "female",
    "phone": "+1 (825) 558-2599"
  },
  {
    "age": 36,
    "name": "Nieves Hill",
    "gender": "male",
    "phone": "+1 (946) 495-3285"
  },
  {
    "age": 30,
    "name": "Dianna Holland",
    "gender": "female",
    "phone": "+1 (948) 406-2941"
  },
  {
    "age": 23,
    "name": "Marsh Robertson",
    "gender": "male",
    "phone": "+1 (903) 413-2132"
  },
  {
    "age": 33,
    "name": "Valenzuela Mcbride",
    "gender": "male",
    "phone": "+1 (998) 499-2074"
  },
  {
    "age": 40,
    "name": "Virginia Michael",
    "gender": "female",
    "phone": "+1 (898) 505-3869"
  },
  {
    "age": 38,
    "name": "Mueller Keller",
    "gender": "male",
    "phone": "+1 (805) 555-3665"
  },
  {
    "age": 37,
    "name": "Madeline Farley",
    "gender": "female",
    "phone": "+1 (954) 446-2747"
  },
  {
    "age": 23,
    "name": "Potter Casey",
    "gender": "male",
    "phone": "+1 (948) 538-3644"
  },
  {
    "age": 24,
    "name": "Melinda Hardy",
    "gender": "female",
    "phone": "+1 (944) 557-2486"
  },
  {
    "age": 34,
    "name": "Monique Carey",
    "gender": "female",
    "phone": "+1 (863) 424-2359"
  },
  {
    "age": 20,
    "name": "Marianne Britt",
    "gender": "female",
    "phone": "+1 (846) 462-2844"
  },
  {
    "age": 37,
    "name": "Guy Langley",
    "gender": "male",
    "phone": "+1 (905) 401-3848"
  },
  {
    "age": 40,
    "name": "Hurst Hogan",
    "gender": "male",
    "phone": "+1 (934) 587-3143"
  }
]

##
cat input.json | jpath '$[0:5].phone'
[
  "+1 (850) 469-2827",
  "+1 (825) 558-2599",
  "+1 (946) 495-3285",
  "+1 (948) 406-2941",
  "+1 (903) 413-2132"
]
cat > /root/answer11.sh << 'EOF'
cat input.json | jpath '$[0:5].phone'
EOF

##
cat input.json | jpath '$[-5:].age'
[
  24,
  34,
  20,
  37,
  40
]
cat > /root/answer12.sh << 'EOF'
cat input.json | jpath '$[-5:].age'
EOF