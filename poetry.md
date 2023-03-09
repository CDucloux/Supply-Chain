## Etape I :

- Ouvrir un terminal **Powershell** avec `ctrl+Alt+ù`

```python
py -m poetry install
```

```python
py -m poetry install --user
```

- La commande ci-dessus installe l'ensemble des *dependencies* - **PREMIERE PARTIE**
- Attention, si vous n'avez pas les autorisations, utilisez `--user` à la fin de la commande !

```python
py -m poetry shell
```

- La commande ci-dessus lance l'environnement virtuel

```python
python -m pip list
```

- Au sein de l'environnement virtuel `python` fonctionne, et cette commande liste les packages installés dans l'environnement

```python
python -m final exemple --help
```

- Enfin, la commande ci-dessus renvoie l'aide associée au package


