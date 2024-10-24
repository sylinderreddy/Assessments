import requests
class Node:
    def __init__(self, node_type, value=None, left=None, right=None):
        self.type = node_type  
        self.value = value  
        self.left = left 
        self.right = right 
    def __repr__(self):
        if self.type == 'operand':
            return f"Operand({self.value})"
        else:
            return f"Operator({self.value})"
class RuleEngine:
    def __init__(self, allowed_attributes):
        self.allowed_attributes = allowed_attributes
    def create_rule(self, rule_string):
        tokens = re.findall(r'\(|\)|AND|OR|>|<|=|\'[^\']*\'|\d+|[a-zA-Z]+', rule_string)
        for token in tokens:
            if token not in ('AND', 'OR', '(', ')', '>', '<', '=', *self.allowed_attributes):
                raise ValueError(f"Invalid token in rule: {token}")
        def parse_expression(tokens):
            stack = []
            while tokens:
                token = tokens.pop(0)
                if token == '(':
                    stack.append(parse_expression(tokens))
                elif token == ')':
                    break
                elif token in ('AND', 'OR'):
                    right = stack.pop()
                    left = stack.pop()
                    node = Node(node_type='operator', value=token, left=left, right=right)
                    stack.append(node)
                elif re.match(r'\w+|>|<|=', token):
                    left = token
                    operator = tokens.pop(0)
                    right = tokens.pop(0)
                    node = Node(node_type='operand', value=f"{left} {operator} {right}")
                    stack.append(node)
            return stack[0]
      return parse_expression(tokens)

  def combine_rules(self, rule_nodes, operator='AND'):
        root = rule_nodes[0]
        for rule in rule_nodes[1:]:
            root = Node(node_type='operator', value=operator, left=root, right=rule)
        return root

  def evaluate_rule(self, node, data):
        if node.type == 'operand':
            # Parse the condition (e.g., 'age > 30')
            attribute, operator, value = node.value.split()
            value = int(value) if value.isdigit() else value.strip("'")
            if operator == '>':
                return data[attribute] > value
            elif operator == '<':
                return data[attribute] < value
            elif operator == '=':
                return data[attribute] == value
        elif node.type == 'operator':
            if node.value == 'AND':
                return self.evaluate_rule(node.left, data) and self.evaluate_rule(node.right, data)
            elif node.value == 'OR':
                return self.evaluate_rule(node.left, data) or self.evaluate_rule(node.right, data)

  def modify_rule(self, node, attribute, new_value):
        if node.type == 'operand':
            if attribute in node.value:
                operator, value = node.value.split()[1], node.value.split()[2]
                node.value = f"{attribute} {operator} {new_value}"
        else:
            if node.left:
                self.modify_rule(node.left, attribute, new_value)
            if node.right:
                self.modify_rule(node.right, attribute, new_value)
        return node

if __name__ == "__main__":
    allowed_attributes = ['age', 'department', 'salary', 'experience']
    engine = RuleEngine(allowed_attributes)

   # Test Case 1: Create individual rules
  print("Test Case 1: Create Individual Rules")
    try:
        rule1 = engine.create_rule("((age > 30 AND department = 'Sales') OR (age < 25 AND department = 'Marketing')) AND (salary > 50000 OR experience > 5)")
        print("Rule 1 AST:", rule1)

   rule2 = engine.create_rule("((age > 30 AND department = 'Marketing')) AND (salary > 20000 OR experience > 5)")
        print("Rule 2 AST:", rule2)
    except ValueError as e:
        print("Error:", e)

  # Test Case 2: Combine rules
   print("\nTest Case 2: Combine Rules")
    combined_rule = engine.combine_rules([rule1, rule2])
    print("Combined Rule AST:", combined_rule)

  # Test Case 3: Evaluate rules
  print("\nTest Case 3: Evaluate Rules")
    data = {"age": 35, "department": "Sales", "salary": 60000, "experience": 3}
    result = engine.evaluate_rule(combined_rule, data)
    print("Evaluation Result for data:", data, "=>", result)

  # Test Case 4: Modify rules
  print("\nTest Case 4: Modify Rule")
  modified_rule = engine.modify_rule(rule1, 'age', '40')
  print("Modified Rule 1 AST:", modified_rule)

  # Test Case 5: Error Handling for Invalid Rules
  print("\nTest Case 5: Error Handling for Invalid Rules")
    try:
        invalid_rule = engine.create_rule("((age > 30 AND dept = 'Sales') OR (age < 25 AND department = 'Marketing'))")
    except ValueError as e:
        print("Error:", e)

  # Test Case 6: Evaluate with different data
  print("\nTest Case 6: Evaluate with Different Data")
    data2 = {"age": 22, "department": "Marketing", "salary": 40000, "experience": 6}
    result2 = engine.evaluate_rule(combined_rule, data2)
    print("Evaluation Result for data:", data2, "=>", result2)

  # Test Case 7: Modify an existing rule to change the department
  print("\nTest Case 7: Modify an Existing Rule")
    modified_rule2 = engine.modify_rule(rule2, 'department', "'HR'")
    print("Modified Rule 2 AST:", modified_rule2)

  # Test Case 8: Evaluate modified rules
  print("\nTest Case 8: Evaluate Modified Rules")
    result3 = engine.evaluate_rule(modified_rule2, data)
    print("Evaluation Result for modified rule 2 with data:", data, "=>", result3)