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
cat input.json | jpath '$[0:3:2]'
[
  "Apple",
  "Microsoft"
]
echo "cat input.json | jpath '$[0,4]'" > /root/answer2.sh 

##
cat input.json | jpath '$[0:5]'
[
  "Apple",
  "Google",
  "Microsoft",
  "Amazon",
  "Facebook"
]
cat input.json | jpath '$[0:5]' > /root/answer3.sh

##
cat input.json | jpath '$[2:7]'
[
  "Microsoft",
  "Amazon",
  "Facebook",
  "Coca-Cola",
  "Samsung"
]
cat input.json | jpath '$[2:7]' > /root/answer4.sh

##
cat input.json | jpath '$[-1:]'
[
  "McDonald's"
]
echo "cat input.json | jpath '$[-1:]'" > /root/answer5.sh

##
cat << 'EOF' > /root/answer5.sh
cat input.json | jpath '$[-1:]'
EOF
